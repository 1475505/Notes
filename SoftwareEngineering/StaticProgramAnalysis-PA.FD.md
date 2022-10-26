# Interprocedural Analysis
> 前面的过程内分析不处理 `method call`，因此需要过程间分析来提高精度。

## call graph(调用图)
![](http://img.070077.xyz//20220927190057.png)
- Method Dispatch of Virtual Calls：双亲委派模型？（由receiver object不断向父类寻找匹配的签名）
## Two Key Functions
![](http://img.070077.xyz//20220927194126.png)
![](http://img.070077.xyz//20220927194102.png)
## Algorithm
![](http://img.070077.xyz//20220927200142.png)
引入ICFG，表示过程间的调用关系。$CFGs + call\& return \ \ edges$
![](http://img.070077.xyz//20220927202353.png)
算法中，为了节省非左值的传递开销，`call site`和`return site`之间的边进行保留，只是`kill`掉左值，更新成被调用函数的返回值。

# Pointer Analysis - Intro
- `may-analysis points-to relations.`，Computes which objects a pointer (variable or field) can point to. 
## Pointers In Java
- Local variable: `x`
- Static field: `A.f`  Sometimes referred as *global variable, like Local variable*
- Instance field: `x.f` Modeled as an object (pointed by x) with a field f
- Array element: `a[i]` 下标访问相关代码块比较复杂，下标一般是变量不是常数等，静态分析的通用做法*忽略下标*，Modeled as an object (pointed by array) with a single field, say arr, which may point to any value stored in array.
## Key Factors
-  Heap Abstraction
-> How to model heap memory?
![](http://img.070077.xyz//20221003213646.png)
-> A technique is Allocation-site. eg. without iteration, only the site(bounded)
- Context Sensitivity: *separate* by different contexts（call）, rather than merge.
- Flow Sensitivity: Actually we dont respect the exection order of stmts, but **maintain a map of points-to relations at the whole program**  for better performance.
- Analysis scope: Whole-program/Demand-driven
## Pointer-Affecting Statements
![](http://img.070077.xyz//20221003222204.png)
![](http://img.070077.xyz//20221003223021.png)

# PA-FD.basic
## Rule（逻辑推导式）
![](http://img.070077.xyz/20221003223855.png)

## Pointer Flow Graph(PFG)
其实就是把`=`理解为`<-`，构造成指针流图。由此转换成传递闭包的问题。比如说一个对于`b`的`Assign`语句，对应的`relation map`就可以沿着有向边传播。
![](http://img.070077.xyz/20221005213345.png)
对于与`field`，其预设的$o_{x} \in pt()$是需要在程序里`Assign`才获得的。Build pointer flow graph (PFG) and Propagate points-to information on PFG is *Mutually dependent*。
![](http://img.070077.xyz/20221005220550.png)
- **Differential propagation** is employed to avoid propagation and processing of redundant points-to information. and no need to propagate them.

# PA-FD with method call
![](http://img.070077.xyz/20221015095024.png)

![](http://img.070077.xyz/20221015110940.png)


# A4：类层次结构分析与过程间常量传播
- 需要使用一个函数的工具，不一定需要从一个实例出发调用，而可能Static utility比如：`CallGraphs.getCallKind(callSite)` 就无需`new`。
- 先捋清楚一些类间关系：
  - Invoke(callsite)： 这相当于`DefStmt`的一个范式。
  - （来自于A5）![](http://img.070077.xyz/20221015213241.png)

    - 对于一个语句 `b = a.m(...)` 来说，`b` 是 `result`，`a.m(...)` 是 `InvokeExp`。
    -   `f(...) {...; b = a.m(...); ...;}`，在这里 `f(...)` 是 `b = a.m(...)` 的 `container`。
    -   如果想获得 `a` 的类型，不是到 `InvokeExp` 里面找，而是到 `MethodRef` 里面找。
> `m` 是一个函数签名，但是 `T` 是方法的集合。那怎么才能通过签名获得方法呢？注意上面说的，不要找到 `container` 去了，答案是通过签名的 `getDeclaringClass()` 获得类信息，再通过类信息获得 `JMethod` 对象。

  - Node：与site的关系是..
- 请注意代码实现与注释规约的一致性。
- 为了DEBUG相应的模块，你可以看一下Assignment的使用方式。


# A5：非上下文敏感指针分析

## 前置知识
- `LoadField` 和 `StoreField` 同时表示实例和静态字段的 load 和 store。这两个类也提供了 `isStatic()` 方法来检查一个 `LoadField/StoreField` 语句是 load/store 静态字段还是实例字段。
- ![](http://img.070077.xyz/20221016095034.png)

 ---
 Thx：https://github.com/RicoloveFeng/SPA-Freestyle-Guidance