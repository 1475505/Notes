## 线段树

线段树是常用的用来维护 **区间信息** 的数据结构，可以在`O(log)`的时间复杂度内实现单点修改、区间修改、区间查询（区间求和，求区间最大值，求区间最小值）等操作。

线段树是一颗近似的完全二叉树，每个节点代表一个区间，节点的权值是该区间的**一个属性**。

例题：[53. 最大子数组和 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/maximum-subarray/)

实例：求区间[l, r]的总和。数组：{10, 11, 12, 13, 14}

我们就可以得到类似于这样的线段树：![](https://oi-wiki.org/ds/images/segt1.svg)

如此，对于区间的查询和数据更新，能够通过迭代，达到比较优秀的性能。

参考：[线段树 - OI Wiki (oi-wiki.org)](https://oi-wiki.org/ds/seg/)

## 梦开始的地方——LCS
