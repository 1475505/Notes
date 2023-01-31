# interface
### interface变量
interface，就是一组抽象方法的集合，里面到底能存什么值呢？如果我们定义了一个**interface的变量，那么这个变量里面可以存*实现*这个interface的任意类型的对象**。

我们怎么反向知道这个变量里面实际保存了的是哪个类型的对象呢？目前常用的有两种方法：
1. Comma-ok断言 `if value, ok := interface.(T); ok`
2. switch测试
```go
switch value := element.(type) {
	case int:
		fmt.Printf("is an int, value is %d\n", index, value)
	case ...
}
// `element.(type)`语法不能在switch外的任何逻辑里面使用
```

### 空interface
所有的类型都实现了空interface。有点类似于C语言的`void`，在我们需要存储任意类型的数值的时候比较有用。

### interface函数参数
可通过定义interface参数，让函数接受各种类型的参数。如对于`fmt.println`：
```go
type Stringer interface {
	 String() string
}
//通过String方法即可实现fmt.Stringer接口
```
则实现此接口的type即可进行print。实现了error接口的对象（即实现了`Error() string`的对象），使用fmt输出时，会调用Error()方法。

# Channel
> 不要通过共享来通信，而要通过通信来共享。

goroutine运行在相同的地址空间，因此访问共享内存必须做好同步。那么goroutine之间如何进行数据的通信呢，Go提供了一个很好的通信机制channel。
```Go
ch := make(chan interface{}, 5) //buffer可存储5个，其余阻塞
ch <- v    // 发送v到channel ch.
v := <-ch  // 从ch中接收数据，并赋值给v

// 可以像操作slice或者map一样操作buffered channel
func produce (n int, ch chan interface{}) {
	for i := 0; i < n; i++ {
		ch <- i
	}
	// 生产者确实不再任何发送数据了，或者想显式的结束range循环之类的可以关闭channel。消费者关闭容易引起panic
}

func consume (ch chan interface{}) {
	for i := range ch { // 读取，直到该channel被显式的close
		fmt.Println(i)
		// 在消费方可以通过语法`v, ok := <-ch`测试channel是否被关闭
	}
}
```

### Select
Go里面提供了一个关键字`select`，通过`select`可以监听channel上的数据流动。默认是阻塞的。当多个监听的channel都准备好的时候，随机选择一个处理。

```go
func fibonacci_consume(c, quit chan int) {
	x, y := 1, 1
	for {
		select {
		case c <- x:
			x, y = y, x + y
		case <-quit:
			return
		case <- time.After(5 * time.Second):
			fmt.println("设置timeout, 避免整个程序进入阻塞")
		default:
			// 当c阻塞的时候执行这里,不再阻塞等待
		}
	}
}
```

# GC

## 追踪式
**程序中运行\所需的数据，一定是栈、数据段这些根节点追踪得到的数据**，因此通过可达性分析，三色抽象如下：
- 白色：init
- 灰色：直接追踪到的root节点，追踪完成后为黑色
- 黑色：存活（可达）数据，追踪到的节点标记为灰色

可见，白色的都是垃圾。基于此：
1. **标记-回收算法**：简单，会导致较多的堆内存垃圾；根据`BiBOP`(*Big Bag of Pages*)的思想，可以规格化分配内存块、统一管理相同内存大小的块。
2. **标记-整理算法**：移动非垃圾数据，使其紧凑地放在内存中。拷贝开销大。
3. **复制式回收**：分区`From` `To`，程序执行时使用`From`，垃圾回收时复制有用的数据到`To`，回收`From`，并交换`From`和`To`空间。降低了堆内存可用量。
4. **分代回收**：基于弱分代假说（参考JVM），数据划分为新生代和老年代，对新生代和老年代执行不同的GC策略。

> 引用计数：**每次对数据对象进行计数**，此方案维护开销较大，且循环引用没回收到。

## 增量式垃圾回收

在三色抽象中，如果不`stop-the-world`，GC会误判黑色对象对白色对象的新引用（同时无灰色对象引用该白色对象）。提出两个不变式：
1. 强三色不变式：**不允许黑色对象对白色对象的新引用** 
2. 弱三色不变式：**如果黑色对象对白色对象新引用，也存在灰色对象引用该白色对象**。
通过读写屏障维持不变式。
![image.png](http://img.070077.xyz/20230120013556.png)

## 混合屏障
- 插入写屏障：结束时需要STW来重新扫描栈，标记栈上引用的白色对象的存活；  
- 删除写屏障：回收精度低，GC开始时STW扫描堆栈来记录初始快照，这个过程会保护开始时刻的所有存活对象。  
  
Go V1.8版本引入了混合写屏障机制（hybrid write barrier），避免了对栈re-scan的过程，极大的减少了STW的时间。结合了两者的优点：

GC时，栈上的新旧对象都标记为黑色，被删除/添加的对象标记为灰色。


## 多核并行垃圾回收
![image.png](http://img.070077.xyz/20230120013848.png)
