# 数据结构
## String

`<data []Byte, len int>`

#### UTF-8编码
![](http://img.070077.xyz/20221216204612.png)

## Slice

`<data []T, len int, cap int>`

#### 扩容规则
1. 预估扩容后容量
<img src="http://img.070077.xyz/20221216204925.png"/>
2. 分配内存，匹配合适的内存规格

## Map
#### 哈希表
![](http://img.070077.xyz/20221216205314.png)
（使用渐进式rehash进行扩容）

上图中右上角的结构即为`bmap`，上方是`topHash`值，接下来的分别是`[]keys, []values`

#### 扩容规则
![](http://img.070077.xyz/20221216205546.png)

# 闭包

Go 语言中函数是头等对象，可以作为参数传递，实际上传递的是指针`FunctionValue`。
```go
type funcva struct {
	fn uintptr
}
```

下图是一个闭包和变量逃逸的例子：
![](http://img.070077.xyz/20221227230244.png)

怎么找到捕获列表？
通过寄存器存储的`funcval`地址加上偏移。

# 方法
方法本质上就是普通的函数，方法接收者即为第一个参数。（Go语言传参是值拷贝，不过非字面量有语法糖）

![](http://img.070077.xyz/20221227230857.png)
（二者其实都是Function Value）

# Defer

Go1.12 的 `runtime.g`模型表征`defer`的注册（`deferproc`），在`runtime.deferreturn()`语句进行执行。内置`defer`池，满了再借助堆分配。执行时，不在捕获列表的变量会拷贝到`defer`结构体进行保存，否则才读取堆的结果。
![](http://img.070077.xyz/20221228013117.png)

Go 1.14 采用`df`变量，按位表示是否需执行和被执行（需要实时条件判断等情况），并使用插入代码的方式进行`defer`机制。循环中的`defer`依然需要使用1.12版本的链表机制。
![](http://img.070077.xyz/20221228013202.png)


```go
type defer struct {
	siz int32  //参数和返回值占用的字节数
	started bool //是否已经执行
	heap bool //是否为堆分配（since 1.13）
	openDefer bool //是否为
	
	sp uintptr //调用者栈指针，判断defer是哪个（层）函数调用的
	pc uintptr //返回地址
	
	fn *funcval
	
	_panic *_panic //指向导致panic的_panic结构体
	link *_defer  //NEXT _defer

	fd unsafe.Pointer
	varp uintptr
	framepc uintptr //三个参数用于栈扫描，找到未注册到链表的defer函数
// 接下来的siz内存为注册时拷贝的参数
}
```

# Panic/Recover

`_panic`链表也在runtime里，和defer链表类似。
```go
type panic struct {
	argp unsafe.Pointer //defer的参数空间地址
	arg interface{}    //panic的参数
	link *_panic     //to earlier panic
	recovered bool   //recover函数控制
	aborted bool
}
```

`panic`后执行（非注册）的`defer`，`defer`中的`panic`会指向该`panic`。执行方式：先标记，后释放，目的是为了终止之前发生的panic。也就是说，执行时再次发生`panic`，插入新panic，而defer中的`_panic`不是它，标记对应的`defer._panic`为已终止。

每个defer函数执行完，会检查`panic.recovered`，移除已恢复的panic，保留对应的defer的sp和pc后移除该defer，根据保存的信息恢复栈帧和指令地址。

例子（defer链表会被执行完，panic不一致会恢复栈帧）：
![](http://img.070077.xyz/20221228015405.png)

# 类型系统

## 类型元数据
每种类型的类型元数据都是全局唯一的。

![](http://img.070077.xyz/20221228020030.png)

## 接口
```go
// runtime.eface。空接口
type eface {
	_type *_type //动态类型，指向类型元数据
	data unsafe.Pointer //动态值
}
```

![](http://img.070077.xyz/20221228020540.png)

`<接口类型, 动态类型> -> itab缓存`哈希表`runtime.itabTableType`，用于存储和查询iTab信息，查找哈希值为`iface.hash ^ _type.hash`

## 类型断言
类型断言作用在接口值之上。

上方的iTab缓存哈希表，对于不存在的类型断言，也会缓存占坑，设置`fun[0]=0`表示不存在。

![](http://img.070077.xyz/20221228021927.png)


-   空接口类型断言实现流程：空接口类型断言实质是将`eface`中`_type`与要匹配的类型进行对比，匹配成功在内存中组装返回值，匹配失败直接清空寄存器，返回默认值。
-   非空接口类型断言的实质是 iface 中 `*itab` 的对比。`*itab` 匹配成功会在内存中组装返回值。匹配失败直接清空寄存器，返回默认值

## reflect

使用`reflect.TypeOf(t)`方法暴露类型元数据。调用参数 t 是编译期处理值拷贝时的**临时拷贝**的地址。利用`ValueOf`方法显式把参数指向的变量逃逸到堆上，实现修改，通过`Elem`得到`reflect.Value`，结合逃逸的对象进行修改。

# GPM

[GO scheduler-ProcessOn](https://www.processon.com/mindmap/604ef3947d9c087fe253e1d4)

Go 语言中，协程对应的数据结构是`runtime.g`，工作线程对应的数据结构是`runtime.m`。为避免全局队列调度加锁的性能损失，g分配到本地队列（满了才留在全局队列），由`p`持有，m执行。
![](http://img.070077.xyz/20221228145613.png)





---

参考资料：
[【幼麟实验室】Golang合辑_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hv411x7we/?spm_id_from=333.999.0.0&vd_source=1e6ac4250e3c54ed91e0fe2b40f533ca)