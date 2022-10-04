# Interprocedural Analysis
> 前面的过程内分析不处理`method call`，因此需要过程间分析来提高精度。

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
- Context Sensitivity: *separate* by different contexts, rather than merge.
- Flow Sensitivity: Actually we dont respect the exection order of stmts, but **maintain a map of points-to relations at the whole program**  for better performance.
- Analysis scope: Whole-program/Demand-driven
## Pointer-Affecting Statements
![](http://img.070077.xyz//20221003222204.png)
![](http://img.070077.xyz//20221003223021.png)

# PA-FD.basic
## Rule（逻辑推导式）
![](http://img.070077.xyz/20221003223855.png)



# A4：类层次结构分析与过程间常量传播
- 需要使用一个函数的工具，不一定需要从一个实例出发调用，而可能Static utility比如：`CallGraphs.getCallKind(callSite)` 就无需`new`。
- 先捋清楚一些类间关系：
  - Invoke(callsite)
  - InvokeExp
  - Node
 