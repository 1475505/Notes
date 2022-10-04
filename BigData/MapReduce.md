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

# Go
## 同步原语
- 闭包：捕获了外部变量的方法体。如：Go中匿名函数。
经常使用WaitGroup在父 goroutine 中阻塞地等待一组子 goroutine 的完结。
```go
func main() {
  var wg sync.WaitGroup
  for i := 0; i < 5; i++ {
    wg.Add(1)
    go func(x int) {
      sendRPC(x)
      wg.Done()
    }(i)
  }
  wg.Wait()
}

func sendRPC(i int) {
  println(i)
}
```

- 线程间同步：一个 goroutine `mu.Unlock` 会唤醒另外的 goroutine 正在阻塞 `mu.Lock`，保证主线程对 `done` 的修改一定能够被子线程看到。回忆下`Java`的`Happen-before`
```go
var done bool
var mu sync.Mutex

func main() {
  println("started")
  go periodic()
  time.Sleep(5 * time.Second) // wait for a while so we can observe what ticker does
  mu.Lock()
  done = true
  mu.Unlock()
  println("cancelled")
  time.Sleep(3 * time.Second) // observe no output
}

func periodic() {
  for done{
    println("tick")
    time.Sleep(1 * time.Second)
    mu.Lock()
    if done {
      return
    }
    mu.Unlock()
  }
}
```

## 锁
- 对于数据竞争，加锁来保护所有对共享变量的访问。
```go
func main() {  
  counter := 0  
  var mu sync.Mutex  
  for i := 0; i < 1000; i++ {  
    go func() {  
      mu.Lock()  
      defer mu.Unlock()  
      counter = counter + 1   // crutical section
    }()  
  }  
  
  time.Sleep(1 * time.Second) // 这里用 WaitGroup 更为稳妥，等待1s只能保证大概率正确。  
  mu.Lock()  
  println(counter)  
  mu.Unlock()  
}
```
- 保护不变量
```go
func main() {
  alice, bob := 10000, 10000
  var mu sync.Mutex

  total := alice + bob

  go func() {
    for i := 0; i < 1000; i++ {
      mu.Lock()
      alice -= 1
      mu.Unlock()
      mu.Lock()   // violate atomic limit below
      bob += 1
      mu.Unlock()
    }
  }()

  start := time.Now()
  for time.Since(start) < 1*time.Second {
    mu.Lock()
    if alice+bob != total {
      fmt.Printf("observed violation, alice = %v, bob = %v, sum = %v\n", alice, bob, alice+bob) // # violation
    }
    mu.Unlock()
  }
}
```
出借和借钱应该是一个原子性操作，因此需要使用锁整个包裹起来。当获取锁进入临界区后，可能会破坏不变性，但是只需要在释放锁前恢复即可。

## 条件变量

```go
func main() {
  rand.Seed(time.Now().UnixNano())

  count := 0
  finished := 0
  var mu sync.Mutex

  for i := 0; i < 10; i++ {
    go func() {
      vote := requestVote()
      mu.Lock()
      defer mu.Unlock()
      if vote {
        count++
      }
      finished++
    }()
  }

  for { // busy wait
    mu.Lock()

    if count >= 5 || finished == 10 { 
      break
    }
	// a pattern to optimize: sleep some time..
    mu.Unlock()
  }
  
  mu.Lock()
  if count >= 5 {
    println("received 5+ votes!")
  } else {
    println("lost")
  }
  mu.Unlock()
}

func requestVote() bool {
  time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
  return rand.Int() % 2 == 0
}

```

上面的代码中，对于`count`的判断，占用了100%的CPU。对于需要一定条件的并发设计，可以使用条件变量。这个方式类似于【信号】，即锁和条件关联了起来。

```go
func main() {
  rand.Seed(time.Now().UnixNano())

  count := 0
  finished := 0
  var mu sync.Mutex
  cond := sync.NewCond(&mu)

  for i := 0; i < 10; i++ {
    go func() {
      vote := requestVote()
      mu.Lock()
      defer mu.Unlock()
      if vote {
        count++
      }
      finished++
      cond.Broadcast()
    }()
  }

  mu.Lock()
  for count < 5 && finished != 10 {
    cond.Wait() // 1. release the mu 2. wait Broadcast() 3. try to acquire the mu again
  }
  if count >= 5 {
    println("received 5+ votes!")
  } else {
    println("lost")
  }
  mu.Unlock()
}
```

## 管道
管道不是队列，会阻塞，也是同步原语。
```go
func main() {
  c := make(chan bool)
  go func() {
    time.Sleep(1 * time.Second)
    <-c
  }()
  start := time.Now()
  c <- true // blocks until other goroutine receives
  fmt.Printf("send took %v\n", time.Since(start))
}
```
带缓冲的 channel 类似一个同步队列，建议在实验中通过共享变量 + 锁来进行多线程间消息的同步和互斥。
```go
func main() {
  done := make(chan bool)
  for i := 0; i < 5; i++ {
    go func(x int) {
      sendRPC(x)
      done <- true
    }(i)
  }
  for i := 0; i < 5; i++ {
    <-done
  }
}
// 可以发挥类似 `WaitGroup` 的作用

func sendRPC(i int) {
  println(i)
}
```


---
参考：
https://laike9m.com/blog/huan-zai-yi-huo-bing-fa-he-bing-xing,61/