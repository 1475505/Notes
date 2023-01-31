
Redis对象：
```c
// src/redis.h
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:24; /* lru time (relative to server.lruclock) */
    int refcount;  // see lifecycle below。 当为零时，对象被销毁，
    void *ptr;
} robj;
```

我们通过`refcount`的变化看看其生命周期。首先是来自client的请求（to args）：
1. set
   对Key和Value的refcount++
2. get
   对Key的refcount++，两条通道
   - `freeClientArgv`：refcount(key)--
   - `sendApplytoClient`：refcount(value)++, 待replyList处理时，refcount(value)--
> 因为redis的场景不会出现循环引用的情况，可以使用引用计数法进行GC

# SDS

## 内部结构

`Simple Dynamic String`是Redis字符串的内部实现。相较于C语言的`char*`，支持了以下Feature:
- 以`O(1)`的时间复杂度获取长度。
- 支持`\0` 字符，以兼容二进制数据的保存（但是 SDS 为了兼容部分 C 语言标准库的函数， SDS 字符串结尾还是会加上 `\0` 字符）
- 避免缓冲区溢出的风险。
```c
struct SDS<T> {
	T capacity;//容量，修改时可以直接判断是否会越界。当判断出缓冲区大小不够用时，Redis 会自动将扩大 SDS 的空间大小。
	T len;//当前串长度。
	Byte flags;//特殊标志位。一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64。
	Byte[] buf;//内容
```

Redis 一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64。**区别在于，它们数据结构中的 len 和 capacity 成员变量的数据类型（位数）不同，为了能灵活保存不同大小的字符串**。

> 除了设计不同类型的结构体，Redis 在编程上还使用了专门的编译优化来节省内存空间，即在 struct 声明了 `__attribute__ ((packed))` ，它的作用是：**告诉编译器取消结构体在编译过程中的优化对齐，按照实际占用字节数进行紧凑对齐**。

## 扩容规则
- 如果所需的 sds 长度**小于 1 MB**，那么最后的扩容是按照**翻倍扩容**来执行的，即 2 倍的newlen
- 如果所需的 sds 长度**超过 1 MB**，那么最后的扩容长度应该是 newlen **+ 1MB**。

## 内部编码（encoding）

![](http://img.070077.xyz/20221225161845.png)
1. `int`：如果一个字符串对象保存的是整数值，并且这个整数值可以用`long`类型来表示，那么字符串对象会将整数值保存在字符串对象结构的`ptr`属性里面（将`void*`转换成 long），并将字符串对象的编码设置为`int`。
2. `embstr`：串长度不大于44（Redis5.0+），调用**一次内存分配**函数来分配一块**连续的内存空间**来保存`redisObject`和`SDS`。
3. `raw`：串长度大于44；调用**两次内存分配**函数来分别分配**两块空间**来保存`redisObject`和`SDS`。

为什么是44B？因为按64B的单元，16B用于存储对象元信息，3B用于存储字符串元信息，最后还需要一个`\0`标识位。
![](http://img.070077.xyz/20221225162803.png)

> embstr实际上是只读的，执行任何修改命令（如append）时，程序会先将对象的编码从embstr转换成raw，然后再执行修改。

# 字典
![](http://img.070077.xyz/20221225165659.png)

- 哈希函数：默认的哈希函数是`siphash`，快且随机性好。

根据负载因子的值（哈希表已保存结点数量 / 哈希表大小）触发rehash：
- 扩容机制：负载因子 >= 1时，就会开始扩容，新数组是原数组大小的 2 倍。如果正在`bgsave`（COW操作），尽量不去扩容，除非 hash 表已经非常满了（负载因子 >= 5）强制扩容。
- 缩容机制：当 hash 表因为元素逐渐被删除变得越来越稀疏时（负载因子 < 0.1），进行缩容。不会考虑 Redis 是否正在做 bgsave 。

>渐进式rehash机制：
>-   给「哈希表 2」 分配空间；
>- **在 rehash 进行期间，每次哈希表元素进行新增、删除、查找或者更新操作时，Redis 除了会执行对应的操作之外，还会顺序将「哈希表 1 」中索引位置上的所有 key-value 迁移到「哈希表 2」 上**；
>-   随着处理客户端发起的哈希表操作请求数量越多，最终在某个时间点会把「哈希表 1 」的所有 key-value 迁移到「哈希表 2」，从而完成 rehash 操作。

# 快速列表

## 压缩列表
为了节约内存空间使用，Redis对象（List 对象、Hash 对象、Zset 对象）在包含的元素数量较少，或者元素值不大的情况才会使用压缩列表作为底层数据结构。压缩列表是一块连续的内存空间，元素之间紧接着存储，没有任何冗余空隙。

```c
struct ziplist<T> {
	int32 zlbytes;//整个压缩列表占用字节数
	int32 zltail_offset;//最后一个元素距离压缩列表起始位置的偏移量，用于快速定位到最后一个节点
	int16 zllength;//元素个数
	T[] entries;//元素内容列表，依次紧凑存储
	int8 zlend;//标志压缩列表的结束，值恒为0xFF
```
`entry`随着容纳的元素类型不同，也会有不同的结构：
```c
struct entry {
	int<var> prevlen;//前一个entry的字节长度，用于逆遍历。变长的整数，当字符串长度小于 254 （即 OxFE ）时，使用一个字节表示；如果达到或超出 254 时 ，就使用 5 个字节来表示`0xFE + int4`。
	int<var> encoding;//元素类型编码
	optional byte[] content;//(可选)元素内容。对于很小的整数而言，它已经内联到 encoding 字段的尾部了。
```

![](http://img.070077.xyz/20221225172529.png)

## 连锁更新
插入一个新的元素就需要调用 realloc 扩展内存。空间扩展操作也就是重新分配内存，因此**连锁更新一旦发生，就会导致压缩列表占用的内存空间要多次重新分配，这就会直接影响到压缩列表的访问性能**。下图是连锁更新的例子：
![](http://img.070077.xyz/20221225173817.png)

## quicklist 结构设计

**通过控制每个链表节点中的压缩列表的大小或者元素个数，来规避连锁更新的问题。** 因为压缩列表元素越少或越小，连锁更新带来的影响就越小，从而提供了更好的访问性能。为了进－步节约空间，还会对 ziplist 进行 LZF 算法压缩存储，可以选择压缩深度。

```c
typedef struct quicklist {
    //quicklist的链表头
    quicklistNode *head;      //quicklist的链表头
    //quicklist的链表尾
    quicklistNode *tail; 
    //所有压缩列表中的总元素个数
    unsigned long count;
    //quicklistNodes的个数
    unsigned long len;
    // LZF 算法深度       
    int compressDepth;
} quicklist;
```

```c
typedef struct quicklistNode {
    //前一个quicklistNode
    struct quicklistNode *prev;     //前一个quicklistNode
    //下一个quicklistNode
    struct quicklistNode *next;     //后一个quicklistNode
    //quicklistNode指向的压缩列表
    ziplist *zl;              
    //压缩列表的的字节大小
    int32 sz;                
    //压缩列表的元素个数
    int16 count;        //ziplist中的元素个数 
    int2 encoding;      //原生存储还是LZF压缩存储   
} quicklistNode;
```
![](http://img.070077.xyz/20221225175016.png)

quicklist 内部默认单个 ziplist 长度为 8KB `list-max-ziplist-size`，默认的压缩深度是 0 `list-compress-depth`，也就是不压缩。为了支持快速 push/pop 操作，quicklist 的*首尾*两个 ziplist 不压缩，此时压缩深度就是1。如果压缩深度为 2，就表示 quicklist 的首尾*第一个* ziplist 以及首尾*第二个* ziplist 都不压缩。

# listpack

quicklistNode 还是用了压缩列表来保存元素，还是存在连锁更新的问题。Redis 在 5.0 新设计一个数据结构叫 listpack，最大特点是**每个节点不再包含前一个节点的长度**了。结构如下：
![](http://img.070077.xyz/20221225180336.png)

此结构的逆序遍历也采用了新方式：从 tail 开始，取element-total-len，获取元素总长度elem-len，往前移到元素开头，重复此访问步骤。

# 整数集合
整数集合是 Set 对象的底层实现之一。当一个 Set 对象*只包含整数值元素*，并且元素数量不大时，就会使用整数集这个数据结构作为底层实现。
```c
typedef struct intset {
    //编码方式.值为 INTSET_ENC_INT`x`，那么 contents 就是一个 int`x`_t 类型的数组，数组中每一个元素的类型都是 int`x`_t；
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
} intset;
```

如果往一个已有3个int16整数集合INTSET_ENC_INT16中加入一个新元素 65535，这个新元素需要用 int32_t 类型来保存，整数集合要进行*升级操作*。首先需要为 contents 数组扩容，**在原本空间的大小之上再扩容多 80 位（4x32-3x16=80），这样就能保存下 4 个类型为 int32_t 的元素**。
![](http://img.070077.xyz/20221226001829.png)

**不支持降级操作**.

# 跳表
zset 结构体里有两个数据结构：一个是跳表，一个是哈希表。这样的好处是既能进行高效的范围查询，也能进行高效单点查询。

```c
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```

## 内部实现

![](http://img.070077.xyz/20221226002509.png)
跳表节点是怎么实现多层级的呢? 每一层级可以包含多个节点，每一个节点通过指针连接起来，实现这一特性就是靠跳表节点结构体中的**zskiplistLevel 结构体类型的 level 数组**。
```c
typedef struct zskiplistNode {
    //Zset 对象的元素值
    sds ele;
    //元素权重值
    double score;
    //后向指针
    struct zskiplistNode *backward;
  
    //节点的level数组，保存每层上的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span; //跨度
    } level[];
} zskiplistNode;
```

zset 对象要同时保存元素和元素的权重，对应到跳表节点结构里就是 sds 类型的 ele 变量和 double 类型的 score 变量。每个跳表节点都有一个后向指针（`struct zskiplistNode* backward`），指向前一个节点，目的是为了方便从跳表的尾节点开始访问节点，这样倒序查找时很方便。

```c
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

跳表结构里包含了：
-   跳表的头尾节点，便于在O(1)时间复杂度内访问跳表的头节点和尾节点；
-   跳表的长度，便于在O(1)时间复杂度获取跳表节点的数量；
-   跳表的最大层数，便于在O(1)时间复杂度获取跳表中层高最大的那个节点的层数量

## 查询与更新

**Redis 跳表在创建节点时候，会生成范围为`[0-1]`的一个随机数，如果这个随机数小于 0.25（`ZSKIPLIST_P`，相当于概率 25%），那么层数就增加 1 层，然后继续生成下一个随机数，直到随机数的结果大于 0.25 结束，最终确定该节点的层数。** 并没有严格维持相邻两层的节点数量比例为 2 : 1 的情况。

# 基数树
rax 是 Redis 内部比较特殊的一个数据结构，它是一个**按Key**有序字典树（基数树Radix Tree），支持快速地定位、插入和删除操作。个人理解为前缀树变种。

rax 被用在 Redis Stream 结构里面用于存储消息ID，在 Redis Cluster 中被用来记录槽位和 key 的对应关系`slots_to_keys`

```c
struct raxNode {
	int<1> isKey;//是否有key,没有key的是根节点
	int<1> isNull;//没有对应的value,是无意义的中间节点
	int<1> isCompressed;//是否压缩存储
	int<29> size;//叶子节点的数量或者是压缩字符串的长
	(isCompressed)
	byte[] data;//路由键、叶子节点指针、value都在这里
```

![](http://img.070077.xyz/20221226003730.png)

| 节点     | 图示 |
| -------- | ---- |
| 压缩结构 |  ![](http://img.070077.xyz/20221226003817.png)
| 非压缩结构         |  ![](http://img.070077.xyz/20221226003831.png)

# AE循环和命令处理

通过IO多路复用`epoll`监听`fd`：
`ReadQueryFromRequest -> fd -> query -> args -> process cmd -> AddReply -> sendReplyToClient`

```c
void readQueryFromClient(aeEventLoop *el, int fd, void *privdata, int mask) {
    redisClient *c = (redisClient*) privdata;
    int nread, readlen;
    size_t qblen;
    REDIS_NOTUSED(el);
    REDIS_NOTUSED(mask);

    server.current_client = c;
    readlen = REDIS_IOBUF_LEN;
    /* If this is a multi bulk request, and we are processing a bulk reply
     * that is large enough, try to maximize the probability that the query
     * buffer contains exactly the SDS string representing the object, even
     * at the risk of requiring more read(2) calls. This way the function
     * processMultiBulkBuffer() can avoid copying buffers to create the
     * Redis Object representing the argument. */
    if (c->reqtype == REDIS_REQ_MULTIBULK && c->multibulklen && c->bulklen != -1
        && c->bulklen >= REDIS_MBULK_BIG_ARG)
    {
        int remaining = (unsigned)(c->bulklen+2)-sdslen(c->querybuf);

        if (remaining < readlen) readlen = remaining;
    }

    qblen = sdslen(c->querybuf);
    if (c->querybuf_peak < qblen) c->querybuf_peak = qblen;
    c->querybuf = sdsMakeRoomFor(c->querybuf, readlen);
    nread = read(fd, c->querybuf+qblen, readlen);
    if (nread == -1) {
        if (errno == EAGAIN) {
            nread = 0;
        } else {
            redisLog(REDIS_VERBOSE, "Reading from client: %s",strerror(errno));
            freeClient(c);
            return;
        }
    } else if (nread == 0) {
        redisLog(REDIS_VERBOSE, "Client closed connection");
        freeClient(c);
        return;
    }
    if (nread) {
        sdsIncrLen(c->querybuf,nread);
        c->lastinteraction = server.unixtime;
    } else {
        server.current_client = NULL;
        return;
    }
    if (sdslen(c->querybuf) > server.client_max_querybuf_len) { //dummy
        sds ci = getClientInfoString(c), bytes = sdsempty();

        bytes = sdscatrepr(bytes,c->querybuf,64);
        redisLog(REDIS_WARNING,"Closing client that reached max query buffer length: %s (qbuf initial bytes: %s)", ci, bytes);
        sdsfree(ci);
        sdsfree(bytes);
        freeClient(c);
        return;
    }
    processInputBuffer(c);
    server.current_client = NULL;
}
```

```c
void sendReplyToClient(aeEventLoop *el, int fd, void *privdata, int mask) {
    redisClient *c = privdata;
    int nwritten = 0, totwritten = 0, objlen;
    size_t objmem;
    robj *o;
    REDIS_NOTUSED(el);
    REDIS_NOTUSED(mask);

    while(c->bufpos > 0 || listLength(c->reply)) {
        if (c->bufpos > 0) {
            if (c->flags & REDIS_MASTER) {
                /* Don't reply to a master */
                nwritten = c->bufpos - c->sentlen;
            } else {
                nwritten = write(fd,c->buf+c->sentlen,c->bufpos-c->sentlen);
                if (nwritten <= 0) break;
            }
            c->sentlen += nwritten;
            totwritten += nwritten;

            /* If the buffer was sent, set bufpos to zero to continue with
             * the remainder of the reply. */
            if (c->sentlen == c->bufpos) {
                c->bufpos = 0;
                c->sentlen = 0;
            }
        } else {
            o = listNodeValue(listFirst(c->reply));
            objlen = sdslen(o->ptr);
            objmem = zmalloc_size_sds(o->ptr);

            if (objlen == 0) {
                listDelNode(c->reply,listFirst(c->reply));
                continue;
            }

            if (c->flags & REDIS_MASTER) {
                /* Don't reply to a master */
                nwritten = objlen - c->sentlen;
            } else {
                nwritten = write(fd, ((char*)o->ptr)+c->sentlen,objlen-c->sentlen);
                if (nwritten <= 0) break;
            }
            c->sentlen += nwritten;
            totwritten += nwritten;

            /* If we fully sent the object on head go to the next one */
            if (c->sentlen == objlen) {
                listDelNode(c->reply,listFirst(c->reply));
                c->sentlen = 0;
                c->reply_bytes -= objmem;
            }
        }
        /* Note that we avoid to send more than REDIS_MAX_WRITE_PER_EVENT
         * bytes, in a single threaded server it's a good idea to serve
         * other clients as well, even if a very large request comes from
         * super fast link that is always able to accept data (in real world
         * scenario think about 'KEYS *' against the loopback interface).
         *
         * However if we are over the maxmemory limit we ignore that and
         * just deliver as much data as it is possible to deliver. */
        if (totwritten > REDIS_MAX_WRITE_PER_EVENT &&
            (server.maxmemory == 0 ||
             zmalloc_used_memory() < server.maxmemory)) break;
    }
    if (nwritten == -1) {
        if (errno == EAGAIN) {
            nwritten = 0;
        } else {
            redisLog(REDIS_VERBOSE,
                "Error writing to client: %s", strerror(errno));
            freeClient(c);
            return;
        }
    }
    if (totwritten > 0) c->lastinteraction = server.unixtime;
    if (c->bufpos == 0 && listLength(c->reply) == 0) {
        c->sentlen = 0;
        aeDeleteFileEvent(server.el,c->fd,AE_WRITABLE);

        /* Close connection after entire reply has been sent. */
        if (c->flags & REDIS_CLOSE_AFTER_REPLY) freeClient(c);
    }
}
```