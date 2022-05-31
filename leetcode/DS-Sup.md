# 线段树

线段树是常用的用来维护 **区间信息** 的数据结构，可以在`O(log)`的时间复杂度内实现单点修改、区间修改、区间查询（区间求和，求区间最大值，求区间最小值）等操作。

线段树是一颗近似的完全二叉树，每个节点代表一个区间，节点的权值是该区间的**一个属性**。也可以理解成：*树状数组*。

例题：[53. 最大子数组和 ](https://leetcode-cn.com/problems/maximum-subarray/)

实例：求区间[l, r]的总和。数组：{10, 11, 12, 13, 14}

我们就可以得到类似于这样的线段树：

```c++
void build(int s, int t, int p) { // 对 [s,t] 区间建立线段树,当前根的编号为 p
	if (s == t) { d[p] = a[s]; return; } 
	int m = s + ((t - s) >> 1); // 如果写成 (s + t) >> 1 可能会超出 int 范围
	build(s, m, p * 2), build(m + 1, t, p * 2 + 1); 
	// 递归对左右区间建树 d[p] = d[p * 2] + d[(p * 2) + 1]; }
```
![](https://oi-wiki.org/ds/images/segt1.svg)
如此，对于区间的查询和数据更新，能够通过迭代，达到比较优秀的性能。

- 通过非叶子节点的“懒惰标记”，延迟对节点信息的更改，从而减少可能不必要的操作次数。

![](https://oi-wiki.org/ds/images/segt4.svg)

```c++
// 区间修改 - 增加值
void update(int l, int r, int c, int s, int t, int p) { // [l, r] 为修改区间, c 为被修改的元素的变化量, [s, t] 为当前节点包含的区间, 为当前节点的编号 
	if (l <= s && t <= r) { d[p] += (t - s + 1) * c, b[p] += c; return; } 
	// 当前区间为修改区间的子集时直接修改当前节点的值,然后打标记,结束修改 
	int m = s + ((t - s) >> 1); 
	if (b[p] && s != t) { // 如果当前节点的懒标记非空,则更新当前节点两个子节点的值和懒标记值 
	d[p * 2] += b[p] * (m - s + 1), d[p * 2 + 1] += b[p] * (t - m); 
	b[p * 2] += b[p], b[p * 2 + 1] += b[p]; // 将标记下传给子节点
	b[p] = 0; // 清空当前节点的标记 
	} 
	if (l <= m) update(l, r, c, s, m, p * 2); 
	if (r > m) update(l, r, c, m + 1, t, p * 2 + 1); 
	d[p] = d[p * 2] + d[p * 2 + 1]; 
}
```

参考：
[树状数组 - OI Wiki (oi-wiki.org)](https://oi-wiki.org/ds/fenwick/)
[线段树 - OI Wiki (oi-wiki.org)](https://oi-wiki.org/ds/seg/)

如果你已经对线段树足够熟悉了，可以练习[307. 区域和检索 ](https://leetcode-cn.com/problems/range-sum-query-mutable/)（[视频讲解](https://www.youtube.com/watch?v=rYBtViWXYeI))


## 二维前缀和/积分图

[304. 二维区域和检索 - 矩阵不可变 ](https://leetcode-cn.com/problems/range-sum-query-2d-immutable/)

数学基础：

![](![](http://img.070077.xyz/202205060423815.png)
<img src="https://assets.leetcode-cn.com/solution-static/304/1.png" alt="fig1" onerror="this.src='data:image/svg+xml,%3Csvg height=\'150\' viewBox=\'0 0 150 150\' width=\'150\' xmlns=\'http://www.w3.org/2000/svg\'%3E%3Cpath d=\'m2465 2286.42347-18.95363-18.92555-50.0112 43.79935-24.62708-24.5906-33.41155 24.5906-22.99654-17.22567v-73.0716c0-2.20914 1.79086-4 4-4h142c2.20914 0 4 1.79086 4 4zm-122-25.59081c5.52285 0 10-4.47052 10-9.98518 0-5.51467-4.47715-9.98519-10-9.98519s-10 4.47052-10 9.98519c0 5.51466 4.47715 9.98518 10 9.98518zm122 40.89296v61.27438c0 2.20914-1.79086 4-4 4h-142c-2.20914 0-4-1.79086-4-4v-53.62625l22.99654 17.22567 33.41155-24.5906 24.62708 24.5906 50.0112-43.79935z\' fill=\'%23eee\' fill-rule=\'evenodd\' transform=\'translate(-2315 -2217)\'/%3E%3C/svg%3E'; ">

代码实现如下：

```c++
// TO get DP
    sum.resize(n+1, vector<int>(m+1,0));// 注意预处理！
    for(int i = 1; i <= n; i++) 
        for(int j = 1; j <= m; j++)
            sum[i][j] = sum[i-1][j] + sum[i][j-1] - sum[i-1][j-1] + matrix[i-1][j-1];

//TO get a[x1:x2][y1:y2]
    x1++; y1++; x2++; y2++;
    ans = sum[x2][y2] - sum[x1-1][y2] - sum[x2][y1-1] + sum[x1-1][y1-1];

```

可参考：[前缀和 & 差分 - OI Wiki](https://oi-wiki.org/basic/prefix-sum/)

学习完前缀和后，可以练习[560. 和为 K 的子数组 ](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

# 单调栈/单调队列

[单调栈 - OI Wiki](https://oi-wiki.org/ds/monotonous-stack/)

将一个元素插入单调栈时，为了维护栈的单调性，需要在保证将该元素插入到栈顶后整个栈满足单调性的前提下弹出元素。**这个弹出的元素，是最新的破坏单调性质的元素**。或者说，栈是个暂存区。

其实只需要填空实现：我们发现了，只要是遇到了相邻两点单调递 *（）* 破坏了单调性质，根据当前栈顶和当前遍历的元素值更新答案。

# 跳表

跳表是对有序链表的改进。跳表的期望空间复杂度为O(n) ，跳表的查询，插入和删除操作，期望时间复杂度都为 O(logn)

- 每层以一定的概率筛选出向上一层的元素。
- 查找时则从顶层开始找。
- 增删则通过伪随机数控制高度。

![](![](http://img.070077.xyz/202205010358042.png)
![](http://img.070077.xyz/202205010358221.png)


# 字典树

如题。

# 哈希

哈希表的查询时间复杂度为什么是O(1)？

### HashCode

数组下标的计算方法是“ `(n - 1) & hash`”。（n 代表数组长度）且HashMap 的长度是 2 的幂次。

**这个算法应该如何设计的？**

取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作，采用二进制位操作 &，相对于%能够提高运算效率。

## 哈希示例

值得提醒的是有哈希并不显式地展示哈希表。如[442. [1, n]数组中重复的数据 ](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/)，即**就地哈希**思想。[Leetcode-offer1#JZ35：复制盲盒链表@就地哈希](Leetcode-offer1.md#JZ35：复制盲盒链表@就地哈希) 中也是使用了*链表原地复制* 的就地方式。

