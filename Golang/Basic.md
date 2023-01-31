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

如何确保某个类型实现了某个接口的所有方法呢？一般可以使用`var _Person = (*Student)(nil)`进行检测，如果实现不完整，编译期将会报错。


接口不仅能作为函数的参数，还能作为结构体的属性。

## 面向对象

> 为什么需要interface？
> 如若一个类耦合太多的功能，内聚度就不够。不如按功能分出接口，使得多个业务代码不用耦合在一个类里。这是**开闭原则**的体现。（[O]一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。在修改需求的时候，应该尽量通过扩展来实现变化）

**依赖倒转原则**(D)：class 应该依赖接口和抽象类而不是具体的类和函数。

| 反例 | ![](http://img.070077.xyz/20230129023128.png) |
| ---- | ---- |
| 正例 | ![](http://img.070077.xyz/20230129022502.png)

我们在设计一个系统的时候，将模块分为3个层次，抽象层、实现层、业务逻辑层。将抽象层的模块和接口定义出来，这里就需要了`interface`接口的设计。


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

# Goroutine与协程
大佬的观点：Goroutine是用户态线程，不是协程。

协程：可暂停和恢复执行的procedure（函数）。有这么一些理解：
1. 本质（核心）是直接调度，也就是**可以控制让出和恢复**。不是用户态线程（运行的载体）。`goroutine`是编程语言内部进行调度的，不支持手动控制执行流（`yield`）。
2. 协程可以作为IO的载体，以解决IO的不确定性。既然是解决不确定性问题，则可以理解为**异步框架**，优势是好写。
3. `coroutine`可理解为【协函数】，比如C++20的`co_wait`等待事件。

# 控制流体操
```go
func test_recover() {  
	defer func() {  
		fmt.Println("defer func")  
		if err := recover(); err != nil {  
			fmt.Println("recover success")  
		}  
	}()  
  
	arr := []int{1, 2, 3}  
	fmt.Println(arr[4])  
	fmt.Println("after panic")  
}  
  
func main() {  
	test_recover()  
	fmt.Println("after recover")  
}
```

输出结果
```
defer func  
recover success  
after recover
```

> 当 panic 被触发时，控制权就被交给了 defer 。后面的代码不执行。

# Notes
对等C中`void*`的数据类型，就是unsafe.Pointer。
