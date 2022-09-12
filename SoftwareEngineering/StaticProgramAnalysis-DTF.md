# Intro
![](http://img.070077.xyz//20220905121635.png)
- More focus on soundness (false negatives)
- over-approximated static analysis produces false positives

# Intermediate Representation
## 3-address form
![](http://img.070077.xyz//20220905124244.png)
- control flow visible
- compact and uniform
![](http://img.070077.xyz//20220905124941.png)
> eg: Java soot Method Call demo:
> ![](http://img.070077.xyz//20220905133806.png)
> ![](http://img.070077.xyz//20220905133834.png)

## Static Simple Assignment(SSA)
给每一个definition都分配一个新的“变量”来用。
优点：弱化控制流的强表示。
缺点：转换成机器码存在问题。

## Control Flow Analysis
CFG nodes：Basic Blocks
> BB：*max* instruction set that can be entered only at the first instruction, as out only at last

# Data Flow Analysis - AP
## Preliminaries
- 状态机转换：
![](http://img.070077.xyz//20220910100454.png)
- 形式化描述：
![](http://img.070077.xyz//20220910102219.png)
- 基本转换函数
$$OUT(B) = gen_B \cup (IN(B)-kill_b)$$
## Reaching Definition Analysis
![](http://img.070077.xyz//20220910105743.png)
	此算法的灵魂在于：`IN[B]`不变，则`OUT[B]`不变。（其他是常量）
	此算法的有穷性表现为：`IN`的变化表示为“更多的流入信息”，信息流在静态程序中是有限的，因此`OUT`为单增的（只会set 1）。
	从循环不变式的角度来说，由于`IN`完全依赖于`OUT`，因此终止条件是安全的不动点。
## Live Variable Analysis
-> 此次定义未被use，则不必存入寄存器（v is dead at p）
![](http://img.070077.xyz//20220912140108.png)
> used: **before** redefinition in B， eg `x = x - 1`
> 为什么用backward?  其实forward也可以，不过更好传递usage和define的消息带到前面去判断，减少迭代次数。(这里OUT不变，IN不变，与上一个算法的依赖相反，是巧合吗？)

## Available Expressions Analysis
指在程序点上，哪个表达式在所有到达此处的路径上已经被计算过了且没有修改。如果被计算，那么可以复用原值。if：
- all paths from the entry to p must pass through the evaluation of x op y
- after the last evaluation of x op y, there is no redefinition(`kill`) of x or y
这是一个`must analysis`(under-approximation, safe)。
| 优化前                                                     | 优化后（使用last evaluation） |
| ---------------------------------------------------------- | ------ |
| ![](http://img.070077.xyz//20220912145838.png)| <img src="http://img.070077.xyz//20220912145538.png"/>     | 

![](http://img.070077.xyz//20220912150537.png)
- 先看kill
- 作为must analysis，初始化的值为全1.
- forward
## Analysis Comparison
![](http://img.070077.xyz//20220912153856.png)

# Data Flow Analysis - FD
