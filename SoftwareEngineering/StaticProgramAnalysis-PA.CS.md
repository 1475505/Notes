# Context Sensitivity
- C.I. mixed objects under different contexts and propagated to other parts of program, causing spurious data flows.
- call-site sensitivity (call-string) represents each context of a method as a chain of call sites.
  - One clone per context(with a signing symbol)
## Context-Sensitive Heap
- OO语言经常修改堆区(对象)，So heap-intensive. 我们给抽象的对象加上上下文。
![](http://img.070077.xyz/20221016170215.png)

The most common choice is to inherit contexts from the method where the object is allocated.

![](http://img.070077.xyz/20221016171545.png)
## Rules
![](http://img.070077.xyz/20221016173411.png)
![](![](http://img.070077.xyz/20221016175300.png)
<img src="http://img.070077.xyz/20221016175300.png"/>
![](http://img.070077.xyz/20221016181225.png)
![](http://img.070077.xyz/20221017194041.png)
