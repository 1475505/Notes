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

# Data Flow Analysis
## Preliminaries
- 状态机转换：
![](http://img.070077.xyz//20220910100454.png)
- 形式化描述：
![](http://img.070077.xyz//20220910102219.png)

## Reaching Definition Analysis
- 转换函数
$$OUT(B) = gen_B \cup (IN(B)-kill_b)$$
![](http://img.070077.xyz//20220910105743.png)
	此算法的灵魂在于：`IN[B]`不变，则`OUT[B]`不变。（其他是常量）
	此算法的有穷性表现为：`IN`的变化表示为“更多的流入信息”，信息流在静态程序中是有限的，因此`OUT`为单增的（只会set 1）。
	从循环不变式的角度来说，由于`IN`完全依赖于`OUT`，因此终止条件是安全的不动点。