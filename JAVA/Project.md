# 项目中间件实践

## 线程池参数

![image.png](https://s2.loli.net/2023/10/02/iGtZ4dqsnyYIDz5.png)


# 长尾延迟

一般来说，我们使用 P95 和 P99 衡量可用性。如果**p99分位的延迟与p95分位的数值差距巨大，就说明该系统存在长尾延迟问题。** 而由于请求的响应时间取决于**最慢的**下游节点的返回，其影响面通常不只是1%。

## 请求容错技术

### backup request

如果请求的耗时超过了pct95的值，那么就触发buckup request，请求副本节点。如果buckup request提前于pct99之前返回，则优化是有效的，这一策略叫做对冲请求（对高延迟的风险对冲），这是以增加了5%请求数为代价的。

为了减少资源浪费，优先返回结果的请求节点可以告知其他节点请求已经被处理，或者组成请求数，通过绑定请求策略告知其他节点取消请求的执行。

> 在一次请求内的优化空间是有限的，会缺少很多有效的全局信息进行决策。
### 分区副本数自适应策略

互联网的马太效应会导致**部分数据的访问会异常高频，成为热点，依旧会造成负载不均衡**，进而产生长尾延迟。针对这一情况，需要对分区**创建副本**，由副本分担读取压力（写的压力通常需要一致性算法如：raft)。同时**热点数据可能会动态变化**，基于上面的情况可以使用一种自适应方法：
- 计算哪些分区具有热点：对GET的埋点记录进行流式分析
- 动态增加副本数量，在热点消退时释放多余的副本

### 延迟熔断策略

当上游发现请求下游某个节点的延迟超过了p99，那么应该将其隔离，在一段时间内不再请求该节点而是请求其他副本，完成请求。通常这里的设计艺术性在于隔离的时长应该如何确定，以及何时触发隔离的间值，通常与tcp的拥塞控制原理差不多。

### 降级策略

- 高扇出的计算任务，允许使用采样/近似/随机等计算手段简化。
- 


