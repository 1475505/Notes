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
