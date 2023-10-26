
Kafka 是一个分布式流式处理平台。

![image.png](https://s2.loli.net/2023/10/23/ixuvNFnWr67KmG1.png)

# 关键功能
  
1. **消息队列**：发布和订阅消息流，建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. **容错的持久方式存储记录消息流**：Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险。  
3. 流式处理平台： 在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。

相比其他消息队列，Kafka主要的优势如下：  
  
1. **极致的性能**：基于 Scala 和 Java 语言开发，设计中大量使用了*批量处理和异步* 的思想，最高可以每秒处理千万级别的消息。  
2. 生态系统兼容性：Kafka 与周边生态系统的兼容性是最好的，尤其在大数据和流计算领域。

## 队列模型
发布订阅模型（Pub-Sub）。

使用主题（Topic） 作为消息通信载体，类似于广播模式；发布者发布一条消息，该消息通过主题传递给所有的订阅者，在一条消息广播之后才订阅的用户则是收不到该条消息的。RocketMQ 的消息模型和 Kafka 基本是完全一样的。唯一的区别是 Kafka 中没有队列这个概念，与之对应的是 Partition（分区）。

![image.png](https://s2.loli.net/2023/10/12/q5huOYojasfAwKQ.png)

1. Broker（代理） : 可以看作是一个独立的 Kafka 实例。多个 Kafka Broker 组成一个 Kafka Cluster。
2. Partition 属于 Topic 的一部分。Topic 下的每个 Partition 只从属于 Consumer Group 中的一个 Consumer。一个 Topic 可以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。一般建议Consumer 和 Partition数量相等，减少出现一个 Consumer 负责多个分区现象。
3. Kafka 通过 Group ID（字符串） 唯一标识 Consumer Group 。

> Kafka 的多分区（Partition）以及多副本（Replica）机制有什么好处呢？  

1. Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。  
2. Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间。

## 保序性

Kafka 中发送 1 条消息的时候，可以指定 topic, partition, key,data（数据） 4 个参数。Kafka 只能为我们保证 Partition(分区) 中的消息有序。消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量（offset）。Kafka 通过偏移量（offset）来保证消息在分区内的顺序性。

## 可靠性

消息不丢失：
- 生产者发送时，设置回调函数，配置重试次数
- 消费者根据自动提交 offset 的开关，控制 offset 的移动在执行前还是执行后(可能重复执行)。
- kafka 存储层：配置集群的消息同步相关参数。

## 可用性

每个 Partition 的副本通常由 1 个 Leader 及 0 个以上的 Follower 组成，Kafka 会尽量将所有的 Partition 以及各 Partition 的副本均匀地分配到整个集群的各个 Broker 上。

![image.png](https://s2.loli.net/2023/10/13/XcNHtZmni6qowVR.png)


# 原理解析

## 消费语义

- Kafka 消费者在默认配置下会进行最多 10 次 的重试，每次重试的时间间隔为 0，即立即进行重试。如果在 10 次重试后仍然无法成功消费消息，则不再进行重试，消息将被视为消费失败。如需自定义重试次数以及时间间隔，只需要在 DefaultErrorHandler 初始化的时候传入自定义的 FixedBackOff 即可。重新实现一个 KafkaListenerContainerFactory ，调用 setCommonErrorHandler 设置新的自定义的错误处理器就可以实现；重写 DefaultErrorHandler 的 handleRemaining 函数，可以加上自定义的告警等操作。
- 死信队列（Dead Letter Queue，简称 DLQ） 是消息中间件中的一种特殊队列。它主要用于处理无法被消费者正确处理的消息。

## reBalance

假如某个  Consumer Group  突然加入或者退出了一个 Consumer，或者  
订阅的 Topic 内的 Partition 发生变更、Topic本身变更等，会发生 “Rebalance”  重平衡 ：
它本质上是一种协议，规定了一个 Consumer Group 下的所有 Consumer 如何达成一致来**分配**订阅 Topic 的每个分区。

## 网络模型

底层基于 Java NIO，采用和 Netty 一样的 Reactor 线程模型。

![image.png](https://s2.loli.net/2023/10/13/NflUSFxuGvPID7A.png)

Kafka 即基于 Reactor 模型实现了多路复用和处理线程池。其设计如下：

![image.png](https://s2.loli.net/2023/10/13/KHaiWRVeQF46wz3.png)

其中包含了一个Acceptor线程，用于处理新的连接，Acceptor 有 N 个 Processor 线程 select 和 read socket 请求，N 个 Handler 线程处理请求并响应，即处理业务逻辑。

## 批量传输与压缩消息

为了让传输消息的次数变得更少、提供吞吐量，实践中，Producer 可以批量发送消息：batch.size和linger.ms（积攒时长）。在 Kafka 中，Kafka 会对消息进行分组，发送消息之前，会先将消息组合在一起形成消息块，然后 Producer 会将消息块一起发送到  Broker 。

![image.png](https://s2.loli.net/2023/10/13/DcPifBAKj2p95bu.png)

## 文件系统

每个 Topic 又可以分为一个或多个分区。每个分区各自存在一个记录消息数据的日志文件。Kafka 充分利用二分法来查找对应 offset 的消息位置。

![image.png](https://s2.loli.net/2023/10/13/j2864dQAcSvikMs.png)
