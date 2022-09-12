## Unix哲学
- 自动化：优先使用工具减轻编码任务
- 快速原型设计：程序单一职责
- 增量式迭代：新工作对应新程序，而非旧程序新特征
- 测试友好与模块化：轻松观察程序进展（如`less`结束流水线）
- 统一接口：期待一个程序的输出成为另一个程序的输入
## MapReduce与分布式文件系统
### What is MapReduce?
![](http://img.070077.xyz/20220823012410.png)

### MapReduce的并行化
- 分布式Unix工具。
- 数据流基于分区实现
# RPC
- Go語言的一大优势在于协程支持：比如可以在后台启动一个goroutine定时执行某件事、周期性的检测什么东西（比如心跳）。由此也衍生出了一些设计风格：[https://go.dev/talks/2012/concurrency.slide](https://go.dev/talks/2012/concurrency.slide)
> **“并发”指的是程序的结构，“并行”指的是程序运行时的状态**（*Different concurrent designs enable different ways to parallelize.*）
> - 并行：就是多个工作单位*同时执行*的意思
> - 并发：程序为多个操作在重叠的时间段内进行提供可能性支持

# GFS
## Guarantees
- consistent: all clients will always see the **same** data, regardless of which replicas they read from.
- defined: consistent and clients will see what the mutation writes in **its** entirety.
![](http://img.070077.xyz//20220911153202.png)
> Record Append: offset by GFS's choosing

--> Specific:
- appending rather than overwriting. (Record Append)
- Single Master with snapshot
  WAL, copy-on-write, checkpoints...
- meta model (Swap in heartbeat)
```
1. ls -> []chunk
2. filename -> []chunk handles
3. chunk handle -> replica location # version
```
# Big Storage
## trade-off
- performance -> sharding
- faults -> tolerance -> replication(REPL) -> in consistency -> low performance

---
参考：
https://laike9m.com/blog/huan-zai-yi-huo-bing-fa-he-bing-xing,61/