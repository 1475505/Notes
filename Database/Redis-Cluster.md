
> Redis集群是一个distribute、fault-tolerant的Redis实现，主要设计目标是达到**线性可扩展性、可用性、数据一致性**。通过 Hash槽、查询路由、节点互联的混合模式实现。

Redis集群本身要解决的是可伸缩问题，以及数据一致、集群可用等一系列问题。前者涉及到了节点的哈希槽的分配(含重分配)，节点的增删，主从关系指定与变更(含自动迁移)这些具体的交互过程；后者则是故障发现，故障转移，选举过程等详细的过程。比如，针对Redis故障宕机造成的缓存雪崩问题，可使用*服务熔断或请求限流机制*，等待 Redis 恢复正常并把缓存预热完后，再解除熔断或限流。最好是可以通过**主从节点的方式构建 Redis 缓存高可靠集群**。

**Redis集群实现的核心思想和思路是什么？** 通过消息的交互（Gossip）实现去中心化(指的是集群自身的实现，不是指数据)，通过Hash槽分配，实现集群线性可拓展。

# 消息交互

>Redis 是个单线程程序，会将每个客户端套接字关联：
>- 一个指令队列，FCFS。
>- 一个响应队列，用于将指令的返回结果回复给客户端，如果队列为空，表示连接空闲，可以将当前的客户端描述符从 write_fds 里面移出 

## Stream

Stream 是 Redis 5.0 版本专门为消息队列设计的数据类型。它有一个消息链表，含唯一消息ID`timestamplnMillis-sequence`和对应的内容。消息是持久化的。而且每个 Stream 都可以挂多个消费组（游标`last_ delivered _id`）。
![](http://img.070077.xyz/20221225024243.png)


#### 常见命令
Stream 消息队列操作命令：
<img src="http://img.070077.xyz/20221225024740.png"/>

-   XADD：插入消息，保证有序，可以自动生成全局唯一 ID；提供个定长长度参数 maxlen，则保持Stream定长，可能丢弃老的消息。
-   XLEN ：查询消息长度；
-   XREAD：用于读取消息，可以按 ID 读取数据；甚至可以当普通队列来使用（不定义消费者，无消息时阻塞等待），如`xread count 2 streams codehole 0-0`
-   XDEL ： 根据消息 ID *懒*删除消息，仅设置标志位，不影响消息总长度；
-   DEL ：删除整个 Stream；
-   XRANGE ：读取区间消息，自动过滤已经删除的消息
-   XREADGROUP：按消费组形式读取消息，进行组内消费。进入消费者的 PEL （正在处理的消息）结构里，处理完毕返回XACK。
-   XPENDING 和 XACK：
    -   XPENDING 命令可以用来查询每个消费组内所有消费者*已读取、但尚未确认*的消息；消费者可以在重启后，用 XPENDING 命令查看这些消息。
    -   XACK 命令用于向消息队列确认消息处理已完成

```
# 0-0 表示从第一条消息开始读取。同一个消费组里的消费者不能消费同一条消息
XGROUP CREATE mymq group1 0-0
# 阻塞读取。命令最后的“$”符号表示读取最新的消息。
XREAD BLOCK 10000 STREAMS mymq $
# 命令最后的参数“>”，表示从第一条尚未被消费的消息开始读取。
XREADGROUP GROUP group1 consumer1 STREAMS mymq >
```

> Redis 消息中间件会不会丢消息？会，以下 2 个场景：
> 1. AOF 持久化配置为每秒写盘，但这个写盘过程是异步的，Redis 宕机时会存在数据丢失的可能
> 2. 主从复制也是异步的，主从切换可能丢数据。

## 通信协议RESP

RESP（Redis Serialization Protocol，Redis序列化协议），将传输的结构数据分为 5 种最小单元类型，单元结束时统一加上回车换行符号 `\r\n`：
1. 单行字符串以`+`符号开头，如`+070077.xyz\r\n`
2. 多行字符串以`$`符号开头，后跟字符串长度。（NULL 用长度为 -1 的多行字符串表示，即`$-1\r\n`；空串用长度为 0 的多行字符串表示`$0\r\n\r\n`）
3. 整数值以`:`符号开头，后跟整数的字符串形式。如`:777\r\n` （如incr命令返回）
4. 错误消息以`-`符号开头。如：`-WRONGTYPE Operation .. \r\n`
5. 数组以`*`号开头，后跟数组的长度。如：`*3\r\n:1\r\n+str\r\n$5\r\nmlstr\r\n`

其实客户端向服务器发送的指令只有多行字符串一种格式，服务器向客户端回复的响应要支持多种数据结构。

```c
void processInputBuffer(redisClient *c) {
    /* Keep processing while there is something in the input buffer */
    while(sdslen(c->querybuf)) {
        /* Immediately abort if the client is in the middle of something. */
        if (c->flags & REDIS_BLOCKED) return;

        /* REDIS_CLOSE_AFTER_REPLY closes the connection once the reply is
         * written to the client. Make sure to not let the reply grow after
         * this flag has been set (i.e. don't process more commands). */
        if (c->flags & REDIS_CLOSE_AFTER_REPLY) return;

        /* Determine request type when unknown. */
        if (!c->reqtype) {
            if (c->querybuf[0] == '*') {  //inline
                c->reqtype = REDIS_REQ_MULTIBULK;
            } else {  //bulked
                c->reqtype = REDIS_REQ_INLINE;
            }
        }

        if (c->reqtype == REDIS_REQ_INLINE) {
            if (processInlineBuffer(c) != REDIS_OK) break;
        } else if (c->reqtype == REDIS_REQ_MULTIBULK) {
            if (processMultibulkBuffer(c) != REDIS_OK) break;
        } else {
            redisPanic("Unknown request type");
        }

        /* Multibulk processing could see a <= 0 length. */
        if (c->argc == 0) {
            resetClient(c);
        } else {
            /* Only reset the client when the command was executed. */
            if (processCommand(c) == REDIS_OK)
                resetClient(c);
        }
    }
}
```

## PubSub（消息多播）

【deparacated】

消息多播：允许生产者只生产一次消息，由中间件负责将消息复制到多个消息队
列，每个消息队列由相应的消费组进行消费；或者是复制到一个普通队列，将多个不同的消费组逻辑串接起来放在一个子系统中。

Redis单独使用 `PubSub` 模块（PublisherSubscriber，发布者/订阅者模式）支持消息多播，不再依赖于 5 种基本数据类型。
```python
# 消费者。
client = redis.StrictRedis()
p = client.pubsub()
# 可以使用Pattern Subscribe模式订阅功能`psubscribe`监听多个主题
p.subscribe("codehole")
# msg = p.get_message() 也可以轮询 get_message 来收取消息。
for msg in p.listen(): # 阻塞监昕消息
    process_msg

# 生产者。必须先启动消费者，然后再执行生产者。如果一个消费者都没有，那么消息会被直接丢弃。
client.publish("codehole","java comes")
```

消息结构：`<pattern, type, channel, data>`
- pattern：订阅的模式。表示当前消息是使用哪种模式订阅到的。如果是通过 subscribe 指令订阅的，那么这个字段就是空。
- type：消息的类型。
  - message：普通消息。
  - subscribe/psubscribe：控制消息，比如订阅指令的反馈。
  - unsubscribe/punsubscribe：取消订阅指令的反馈。
- channel ：当前订阅的主题名称。

# 持久化

Redis 的持久化机制有两种，第一种是 RDB 快照（全量备份），第二种是 AOF 日志（连续的增量备份）：

## RDB 快照

内存数据的二进制序列化形式，在存储上非常紧凑。

Redis 提供了两个命令来生成 RDB 文件，分别是 `save` 和 `bgsave`，他们的区别在于是否后台（*b*ack*g*round，避免阻塞主线程）执行。RDB 文件的加载工作是在服务器启动时自动执行的。可以配置bgsave自动执行的条件，如`save 300 10`表示300 秒之内，对数据库进行了至少 10 次修改。

Redis 使用操作系统`fork`的多进程 COW（Copy On Write）机制来实现快照持久化，这样就可以避开文件写入阻塞线程（文件 IO 操作不能使用多路复用 API）。同时，就算有数据竞争，这里也会采用`fork`出来时的内存数据作为 RDB 数据，屏蔽主线程的修改操作。

## AOF 日志

记录内存数据**修改的指令**日志文本。当 AOF 日志的大小超过所设定的阈值后，由于数据库重启时需要加载 AOF 日志进行指令重放，所以需要定期进行 AOF 重写（日志压缩）。注：不同于WAL，先执行指令才将日志存盘（不会阻塞当前写操作命令的执行）。

如果在将日志内容写入到硬盘时，服务器的硬盘的 I/O 压力太大，会影响*后续*命令的执行。根据`fsync(int fd)`系统调用的时机，因此有三种写回策略：
- **Always**，每次写操作命令执行完后，同步将 AOF 日志数据写回硬盘；
- **Everysec**，每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，然后*每隔一秒* 将缓冲区里的内容写回到硬盘；
- **No**，意味着不由 Redis 控制写回硬盘的时机，转交给操作系统控制写回的时机，由决定何时将内核缓冲区内容写回硬盘。

> 相较于Always，当使用 Everysec 策略时，由于是异步执行 fsync，大 Key 持久化的过程（数据同步磁盘）不会影响主线程。

![](http://img.070077.xyz/20221224034905.png)

AOF 重写机制：读取当前数据库中的所有键值对，然后将每一个键值对用一条命令记录到*新的 AOF 文件*（防止污染原数据），等到全部记录完后，就将新的 AOF 文件替换掉现有的 AOF 文件。
![](http://img.070077.xyz/20221224035153.png)

Redis 的重写 AOF 过程是由后台子进程 `bgrewriteaof` （也是指令）来完成的，这么做可以达到两个好处：
- 子进程进行 AOF 重写期间，主进程可以继续处理命令请求，从而避免阻塞主进程；
- 子进程作为`fork`出来的副本，结合`COW`进行重写。如果此时主进程修改了已经存在 key-value，这个修改就会发生**写时复制COW**，通过**AOF 重写缓冲区**处理数据不一致。即，在重写 AOF 期间，当 Redis 执行完一个写命令之后，它会**同时将这个写命令写入到 AOF 缓冲区和 AOF 重写缓冲区**。

> 如果存在大key，页表的复制也不会小，fork 函数执行时也可能会发生阻塞。

![](http://img.070077.xyz/20221224035923.png)

> 混合持久化
> Redis 4.0起，为解决大实例重放 AOF 日志启动慢的问题，引入**混合持久化**选项：即文件包含`<RDB 快照的内容，和增量（持久化开始到持久化结束的这段时间发生的增量，AOF 重写缓冲区）的 AOF 日志文件>`。

# 主从同步

## 复制

默认是**增量同步**：主节点会将那些对自己的状态产生修改性影响的指令记录在本地内存的*环形缓冲区* repl_backlog_buffer 中，然后异步将 buffer 中的指令同步到从节点。从节点会向主节点反馈当前同步的偏移量。从节点不会进行过期扫描，如果`key`在主节点被过期删除，主节点会在 AOF 文件里增加一条 del 指令模拟同步，这是异步的，**会出现短暂的数据不一致**。

>从节点每隔 1 秒发送 replconf ack{offset} 命令，给主节点上报自身当前的复制偏移量。
>主节点默认每隔 10 秒对从节点发送 ping 命令，判断从节点的存活性和连接状态。

快照同步：如果 Buffer 满了，就会覆盖未被同步的指令。需要同步全量快照。

我们可以使用 `replicaof`（Redis 5.0 之前使用 slaveof）命令形成主服务器和从服务器的关系（这个关系可以是多级的）。然后，主从服务器间的第一次同步的过程可分为三个阶段：
![](http://img.070077.xyz/20221224233951.png)

-   第一阶段是建立连接、协商同步。从服务器就会给主服务器发送 `psync(runID, offset)` 命令，表示要进行数据同步。
-   第二阶段是主服务器同步数据给从服务器；在生成、发送、加载RDB时，写操作命令将写入**replication buffer**。
-   第三阶段是主服务器发送新写操作命令给从服务器。将 replication buffer 缓冲区里所记录的写操作命令发送给从服务器。

主从服务器在完成第一次同步后，双方之间就会维护一个 TCP **长连接**，后续主服务器可以通过这个连接继续将写操作命令传播给从服务器。

>replication buffer 、repl backlog buffer 区别如下：
>1. 出现的阶段不一样
>   - repl backlog buffer 是在增量复制阶段出现，一个主节点只分配一个 repl backlog buffer；
>   - replication buffer 是在全量复制阶段和增量复制阶段都会出现，主节点会给每个新连接的从节点，分配一个 replication buffer；
>2. 当缓冲区满了之后，发生的事情不一样：
>   - 当 repl backlog buffer 满了，因为是环形结构，会直接覆盖起始位置数据;
     - 当 replication buffer 满了，会导致连接断开，删除缓存，从节点重新连接，重新开始全量复制。

## 心跳机制

> 由于网络问题，集群节点之间失去联系。主从数据不同步；重新平衡选举，产生两个主节点（脑裂）。等网络恢复，旧主节点会降级为从节点，再与新主节点进行同步复制的时候，由于从节点会清空自己的缓冲区，所以导致之前客户端写入的数据丢失了。
> 解决方案：当主节点发现从节点下线或者通信超时的总数量小于阈值时，那么禁止主节点进行写数据，直接把错误返回给客户端。配置`min-slaves-to-write`  和`min-slaves-max-lag`，达到：**主节点连接的从节点中至少有 N 个从节点，「并且」主节点进行数据复制时的 ACK 消息延迟不能超过 T 秒**

`Info`指令可以诊断集群的运行情况。如：`info memory` `info`
```
1. Server：服务器运行的环境参数。
2. Clients：客户端相关信息。
3. Memory：服务器运行内存统计数据。如：used_memory_human
4. Persistence：持久化信息。
5. Stats：通用统计数据。如：ops_per_sec,rejected_connections,sync_partial_err
6. Replication：主从复制相关信息。如：repl_backlog_size
7. CPU：CPU 使用情况。
8. Cluster：集群信息。如：connected clients,
9. KeySpace：键值对统计数量信息。
```


# 集群模式

## 单机模式
Redis 主节点以单个节点的形式存在，这个主节点可读可写，上面存储数据全集。通常单机模式为“1主 N 备”的结构。

问题：
- 高并发瓶颈。计算（聚合）操作会严重影响整体吞吐
- 无法自动故障转移（Failover）。故障转移需要“哨兵”Sentinel 辅助
- 仅支持纵向扩容

### 哨兵机制

Sentinel 负责持续监控主从节点的健康。Sentinel 无法保证消息完全不丢失，但是也能尽量保证消息少丢失。哨兵节点主要负责三件事情：**监控、选主、通知**。为减少误判的情况，哨兵在部署的时候会部署多个节点部署成**哨兵集群**（最少需要*三* 台机器来部署哨兵集群），**通过多个哨兵节点一起判断，就可以就可以避免单个哨兵因为自身网络状况不好，而误判主节点下线等情况。**

哨兵会每 10 秒一次的频率向主节点发送 INFO 命令来获取所有从节点的信息。如果主节点或者从节点没有在规定的时间内响应哨兵的 PING 命令，哨兵就会将它们标记为**主观下线**。这个规定的时间是配置项 `down-after-milliseconds` 参数设定的，单位是毫秒。对于*主节点*，若哨兵判断其*主观下线*后，就会向其他哨兵发送`is-master-down-by-addr` 命令进行投票，让它们根据自身和主节点的网络状况进行表决。当这个哨兵的赞同票数达到哨兵配置文件中的 quorum 配置项设定的值后，这时*主*节点就会被该哨兵标记为**客观下线**。

故障转移：
- 选出新主节点，对其发送指令`SLAVEOF no one`。Redis 有个叫 down-after-milliseconds * 10 配置项，即主从节点断连的最大连接超时时间。如果发生断连的次数超过了 10 次，就说明这个从节点的网络状况不好，不适合作为新主节点。过滤后，进行三轮考察：**优先级`slave-priority`、复制进度`master/slave_repl_offset`、ID 号**。
- 当新主节点出现之后，哨兵 leader 会向已下线主节点属下的所有从节点发送 `SLAVEOF` 命令，指向*新主节点*.
- 通过Redis 的发布者/订阅者机制来实现通知客户：主节点已更换。
![](http://img.070077.xyz/20221225003200.png)
- 将旧主节点更新为从节点。当旧主节点重新上线时，哨兵集群就会向它发送`SLAVEOF` 命令，让它成为新主节点的从节点。
![](http://img.070077.xyz/20221225002741.png)

> 哨兵集群中，节点之间是通过 Redis 的发布者/订阅者机制来相互发现的——主节点上有一个名为`__sentinel__:hello`的频道。
> `sentinel monitor <master-name> <ip> <redis-port> <quorum> 

## Redis Cluster

### 切片
Redis 集群实现的基础是*分片*，即将数据集有机的分割为多个片，并将这些分片指派给多个 Redis 实例，每个实例只保存总数据集的一个子集，之间通过一种特殊的二进制协议交互集群信息。将所有数据划分为 16384 个槽位（Hash Slot），通过客户端缓存槽位配置信息及纠正机制（`MOVED slot_id node_ip` ）定位Node。
![](http://img.070077.xyz/20221227224034.png)


槽位号：通过 [CRC16 算法 ](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)计算出 16 bit 值，取最右边的 14 个 bit （%16384）。
> 如果用户的 key 包含 {...} 这个样子的字符串的话, 只有 { 中间的部分 } 会被进行哈希. 这个策略可以满足用户希望强制不同的 key 映射到相同节点的需求(假设没有正在进行 resharding)

在实际应用中一般采用“一致性哈希”算法，在增删节点的时候，可以保证尽可能多的缓存数据不失效。

槽位号路由到具体的 Redis 节点上有两种方案：
-   **平均分配：** 在使用 cluster create 命令创建 Redis 集群时，Redis 会自动把所有哈希槽平均分布到集群节点上。比如集群中有 9 个节点，则每个节点上槽的个数为 16384/9 个。
-   **手动分配：** 可以使用 cluster meet 命令手动建立节点间的连接，组成集群，再使用 cluster addslots 命令，指定每个节点上的哈希槽个数。`redis-cli -h 192.168.1.10 –p 6379 cluster addslots 0,1`
![](http://img.070077.xyz/20221225132449.png)

> 为了克服客户端分片业务逻辑与数据存储逻辑耦合的不足，可以通过 Proxy 将业务逻辑和存储逻辑隔离，**基于代理进行分片**。即，Proxy+Redis-Server。

### 故障处理
Redis Cluster不需要 Sentinel，通过集群内部主节点选举完成故障处理，是一个“自治”的系统。可划分为三大步骤：故障检测、从节点选举以及故障倒换。

1. 故障检测
   集群中的各个节点会通过相互发送消息的方式来交换自己掌握的集群中各个节点的状态信息。
   单点视角：Gossip的pingpong消息。发送 Ping 消息的节点就会将无响应的节点标注为疑似下线状态（Probable Fail，Pfail）。
   集群视角：**超过半数的持有 Slot（槽）的主节点都将某个主节点 X 报告为疑似下线**，那么，主节点 X 将被标记为下线（Fail），并广播出去。
2. 选举
   选举新主节点的算法是基于 Raft 算法的 Leader Election 方法来实现的。
3. 故障倒换
   获胜的从节点将发起故障转移（Failover），角色从 Slave 切换为 Master，并接管原来主节点的 Slots。

## Codis

Redis Cluster 有很多优点，但是，当集群规模超过百节点级别后，Gossip 协议的效率将会显著下降，通信成本越来越高。而且，任何一个被指派 Slot 的主节点故障，在其恢复期间，集群都是不可用的。

Codis 出现在 Redis Cluster 之前，使用简单，自动平衡，提供命令行接口，支持 RESTful APIs。如下：

![](http://img.070077.xyz/20230125043110.png)

Codis 主要由四部分组成：
-   Codis Proxy（`codis-proxy`）：是客户端连接的 Redis 代理服务，它本身实现了 Redis 协议。对于一个业务来说，可以部署多个 Codis Proxy，Codis Proxy 本身是无状态的。
-   Codis Manager（`codis-config`）：是 Codis 的管理工具，支持添加/删除 Redis/Proxy 节点、发起数据迁移等。本身还自带了一个 HTTP Server，会启动一个 Dashboard。
-   Codis Redis（`codis-server`）：是 Codis 项目维护的一个 Redis 分支，加入了对 Slot 的支持和原子的数据迁移指令。（耦合）
-   ZooKeeper：存放数据路由表和 `codis-proxy` 节点的元信息和同步。

## 迁移

Redis 迁移的单位是槽。从源节点获取内容→存到目标节点→从源节点删除内容。迁移过程是同步的，在目标节点执行 restore 指令到源节点删除 key 之间，源节点的主线程会处于阻塞状态，直到 key 被成功删除。
![](http://img.070077.xyz/20221225022457.png)

在迁移过程中，客户端访问的流程会有很大的变化。先尝试访问旧节点，如果数据不存在（可能本身就不存在，或已迁移），返回重定向指令`-ASK targetNodeAddr`，客户端先去目标节点执行一个不带参数的 ASKING 指
令（因为在迁移没有完成之前，按理说这个槽位还是不归新节点管理的，为防止其返回`MOVED`导致重定向循环。此指令不会刷新槽位映射关系表），然后在目标节点再重新执行原先的操作指令，由此实现**槽位迁移感知**。

# Redis安全与可用性

## 指令与端口
- `rename-command keys sudokeysinmyheart`：将某些危险的指令修改成特别的名称，用来避免人为误操作。如果`rename`成空串，就没办法执行`keys`了。
- 配置密码和端口
- SSL 代理。Redis 并不支持 SSL 连接，可以使用 SSL 代理，这也可以用在主从复制上。比如使用`Spiped`：
![](http://img.070077.xyz/20221225114923.png)

## 缓存穿透
当用户访问的数据，**既不在缓存中，也不在数据库中**，导致请求在访问缓存时，发现缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据，没办法构建缓存数据，来服务后续的请求。那么当有大量这样的请求到来时，数据库的压力骤增，这就是**缓存穿透**的问题。应对方案：
1. 限制非法请求。API处理端判断传参是否合理。
2. 缓存空值/默认值。可以针对查询的数据，在缓存中设置一个空值或者默认值`NOT FOUND`。
3. **使用布隆过滤器快速判断数据是否存在**。


![](http://img.070077.xyz/20221225123500.png)

## 大Key
大 key 会带来客户端超时阻塞、网络阻塞、工作线程阻塞、内存分配不均等问题。一般而言，下面这两种情况被称为大 key：
-   String 类型的值大于 10 KB；
-   Hash、List、Set、ZSet 类型的元素的个数超过 5000个；

大Key可以通过以下方式寻找到：
1. 在从节点上（使用 -i 参数控制扫描间隔）执行`redis-cli -h 127.0.0.1 -p6379 -a "password" -- bigkeys` 返回每种类型中最大的那个 bigkey。（但是对于集合类型来说，这个方法只统计集合元素个数的多少，而不是实际占用的内存量）
2. `scan`
3. 解析RDB文件，如工具`rdb dump.rdb -c memory --bytes 10240 -f redis.csv`

如何优雅地删除大Key？
1. 分批次删除。如，对于大hash，使用 `hscan` 命令，每次获取 100 个字段，再用 `hdel` 命令，每次删除 1 个字段；对于大 List，通过 `ltrim` 命令，每次删除少量元素。类似地有`srem` `zremrangebyrank` 等。
2. 异步删除。主要有 4 种场景，默认都是关闭的：
```conf
lazyfree-lazy-eviction no # 表示当 Redis 运行内存超过 maxmeory 时，是否开启 lazy free 机制删除；
lazyfree-lazy-expire no  # 表示设置了过期时间的键值，当过期之后是否开启 lazy free 机制删除；
lazyfree-lazy-server-del no # 有些指令在处理已存在的键时，会带有一个隐式的 del 键的操作，比如 rename 命令，当目标键已存在，Redis 会先删除目标键，如果这些目标键是一个 bigKey，就会造成阻塞删除的问题，此配置表示在这种场景中是否开启 lazy free 机制删除；
noslave-lazy-flush no  # 针对 slave (从节点) 进行全量数据同步，slave 在加载 master 的 RDB 文件前，会运行 flushall 来清理自己的数据，它表示此时是否开启 lazy free 机制删除。
```

# 分布式锁

## 悲观锁
由String基本数据类型实现。只允许被一个客户端占有，先来先占，为了防止中间逻辑异常，再加上过期时间，以避免死锁。可以使用`set`指令，或使用`setnx`(set if not exists)指令配合`expire`加锁，用完了，再调用del指令释放；
```redis
set lock:codehole true ex 5 nx 
# SET lock_key unique_value（客户端唯一标识） NX（仅不存在时set） PX 10000（过期时间10s）
# 在Redis2.8+支持的原子指令，以防断电等造成非原子性。
del lock:codehole
```

如果在加锁和释放锁之间的逻辑执行时间过长，以至于超出了锁的超时限制，临界区的逻辑则无法严格串行执行。
- 方案一：是使用[Lua 脚本](https://redisbook.readthedocs.io/en/latest/feature/scripting.html)保证连续多个指令的原子性执行。
```redis
tag = random.nextint();  #随机数
if redis.set(key, tag, nx=True, ex=5):
	do_something ()
	redis.delifequals(key, tag) ＃假想的del_if_equals指令
```
```Lua
#delifequals
if redis.call("get", KEYS[1]) == ARGV[1] then
	return redis.call("del", KEYS[1])
else
	return 0
end
```
- 方案二：**支持可重入**。对客户端的 set 方法进行包装，使用线程的 Threadlocal 变量存储当前持有锁的计数。精确一点还需要考虑内存锁计数的过期时间，复杂性很高，不建议使用。

> 集群环境，如果主从发生 `failover`，就不安全了。引入[RedLock](https://redis.io/topics/distlock)。
> 加锁时，需要向过半节点发送 `set(key, value, nx=True, ex)`指令，只要过半节点 set 成功，就认为加锁成功。释放锁时，需要向所有节点发送 del 指令。这个算法会损失一定的性能。