# 数组/字符串专题

双指针是在时空复杂度限制下，比较理想的思考方案，滑动窗口、快速选取都可以归到这一领域。

## 滑动窗口

[76. 最小覆盖子串 ](https://leetcode-cn.com/problems/minimum-window-substring/)

思路：使用哈希表计数索引，用哈希表动态维护窗口中所有字符及个数。先不断`j++`扩大窗口，使得窗口内包含T的所有元素，再`i++`缩小窗口，记录下这一流程下的必要最小值后，放弃这个位置，重复上述过程。

[480. 滑动窗口中位数](https://leetcode-cn.com/problems/sliding-window-median/)

JZ41中[295. 数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream/)便可通过大小根堆联动处理，我们可以采取类似的思路。关键是怎么处理删除数据的情况。介绍一个技巧，**懒惰标记技术**。由于堆无法直接删除掉某个指定元素，先欠下这个账，等它出现在堆顶的时候，再删除。对应地，记录`bias`维护small堆元素数目与big堆元素个数差。

> 更令OIer 欣喜的是，[平衡树 - pb_ds](https://oi-wiki.org/lang/pb-ds/tree/)能更方便地解决这一问题。在此略去。

这个思想可以用来尝试[218. 天际线问题 ](https://leetcode-cn.com/problems/the-skyline-problem/)。

## 划分/选取系列

荷兰国旗问题、topK问题在《剑指Offer》笔记中已提及，此处略。

[524. 通过删除字母匹配到字典里最长单词](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)

思路：归并两个有序数组。通过双指针匹配“相邻”时，需要特别注意。

[128. 最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

思路：并查集。其实现可以参考OI-wiki。因为求的是“序列”，能想到哈希表当然就更好了。

## 单调栈系列

[739. 每日温度](https://leetcode-cn.com/problems/daily-temperatures/)

通过单调栈解决。
思路：为了方便计算天数差，我们这里存放下标而非温度本身，因为存放温度本身不便反算下标。类似的题：[901. 股票价格跨度](https://leetcode.cn/problems/online-stock-span/)

[456. 132 模式 ](https://leetcode-cn.com/problems/132-pattern/)

以“3”为基准，记录【左边最小值】和【右边最大值】，后者可通过单调递减栈求出。

```c++
		for j in range(N - 1, -1, -1):
            numsk = float("-inf")
            while stack and stack[-1] < nums[j]:
                numsk = stack.pop()
            if leftMin[j] < numsk:
                return True
            stack.append(nums[j])
```

> 拓展问题：满足132模式的子序列有多少个。
> 甚至有一些有趣的研究。[Stack-sortable permutation - Wikipedia](https://en.wikipedia.org/wiki/Stack-sortable_permutation)

[42. 接雨水 ](https://leetcode.cn/problems/trapping-rain-water/)： 单调递减栈，碰到元素大于栈顶的时出栈算水量。

> [407. 接雨水 II - 力扣（LeetCode）](https://leetcode.cn/problems/trapping-rain-water-ii/)：学霸题，数长方体，遍历找坑法，按行按列数.....

[300. 最长递增子序列 ](https://leetcode.cn/problems/longest-increasing-subsequence/)

## 区间系列

首先，这类题目常出现的操作是，给定一个二维数组（多个起止时间段），而你需要在一维数组（一个时间轴）上进行处理，一个常用操作是[56. 合并区间](https://leetcode.cn/problems/merge-intervals/)。

1.技巧：可以对二维`vector`进行排序，会先以【第一列】进行排序，再排后续列。逆过程相当于：[57. 插入区间](https://leetcode.cn/problems/insert-interval/)。

2.排序后有动态规划、滑动窗口等优化复杂度的思考方向。

3.区间系列题中，还有可能出现数据流输入的情况。

[919 · 会议室 II - LintCode](https://www.lintcode.com/problem/919/description)

这道题可以用【前缀和】【扫描线】两种思路思考。（上面练过）

而且这个时候就不得不提一道经典题目了：

![](http://img.070077.xyz/202205070804953.png)

## 综合

[238. 除自身以外数组的乘积 ](https://leetcode.cn/problems/product-of-array-except-self/)：可以考虑一些区间处理的数据结构
[84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)：接雨水的兄弟版本
[139. 单词拆分](https://leetcode.cn/problems/word-break/)：选取系列
[48. 旋转图像](https://leetcode.cn/problems/rotate-image/)：阴间细节题

# 链表专题

## 反转链表

[206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)

- 非递归写法
```cpp
ListNode *prev = nullptr, *next;
while (head) {
	next = head->next;
	head->next = prev; 
	prev = head;
	head = next;
}
return prev;
```
- 递归写法
```c++
ListNode* reverseList(ListNode* head, ListNode* prev=nullptr) {
	if (!head) return prev;
	ListNode* next = head->next;
	head->next = prev;
	return reverseList(next, head);
}
```

> 链表特别需要注意**考虑每个节点的指针**，拓展题：
> [92. 反转链表 II ](https://leetcode.cn/problems/reverse-linked-list-ii/)
> [25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)
> 关键是分割和连接。


## 环形链表/数组

[环形链表](https://leetcode-cn.com/problems/linked-list-cycle-ii/solution/linked-list-cycle-ii-kuai-man-zhi-zhen-shuang-zhi-/)

1. 快慢指针。`fast`每次走两步，`slow`每次走一步，二者会在环里的某个位置相遇，此时`fast`比`slow`超前若干个环长。即：
   $$f=2s=s+nl$$
2. 得知 $s = nl$ 。记链表头距离环的入口$a$， 则再走$a$步，即可到达环的入口，**因为$a+nl$ 与 $a$ 同位置。**
3. 利用环形的特性，可以写出如下的代码：

```cpp
		if (head == NULL || head->next == NULL) return NULL;
        ListNode* slow = head;
        ListNode* fast = head;
        do{
            slow = slow->next;
            if (fast && fast->next)
                fast = fast->next->next;
            else
                return NULL;
        } while(slow != fast);
        assert(slow == fast);
        slow = head;
        while (slow != fast){
            slow = slow->next;
            fast = fast->next;
        }
        return slow;
```

> 拓展：本题思想可以解决[287. 寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/)。
> 关键是把看作数组的值看作映射。

[503. 下一个更大元素](https://leetcode-cn.com/problems/next-greater-element-ii/)

这是一道典型的单调栈题目，栈中存储的是还没找到下一个更大元素的元素下标。其中的环形思想，可以通过【求模】实现。

## 链表归并

[归并排序链表](https://leetcode-cn.com/problems/sort-list/solution/sort-list-gui-bing-pai-xu-lian-biao-by-jyd/)

关键有两步：
- `cut`：使用快慢指针，找到链表的中点，断开。
- `merge`：交替比较头部，添加完成链表。

从评论区中得到了比较直观的伪代码如下：
```cpp
current = dummy.next;
tail = dummy;
for (step = 1; step < length; step *= 2) {
	while (current) {
		// left->@->@->@->@->@->@->null
		left = current;

		// left->@->@->null   right->@->@->@->@->null
		right = cut(current, step); // 将 current 切掉前 step 个头切下来。

		// left->@->@->null   right->@->@->null   current->@->@->null
		current = cut(right, step); // 将 right 切掉前 step 个头切下来。
		
		// dummy.next -> @->@->@->@->null，最后一个节点是 tail，始终记录
		//                        ^
		//                        tail
		tail.next = merge(left, right);
		while (tail->next) tail = tail->next; // 保持 tail 为尾部
	}
}
```

归并排序是理解递归的经典材料，吃透了方可在树专题中得心应手。

[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

可以类似于归并排序进行两两合并，也可以把**所有的链表存储在一个优先队列**中，每次提取所有链表头部节点值最小的那个节点，直到所有链表都被提取完为止。

# 动态规划专题

本专题内所有题目均可拓展：输出路径。

## 字符串篇

> 万物皆是~~dp~~ LCS...

[516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

你应该记得......那是某个秋天，王晓茹的课上，循序渐进引出的例题......反转字符串+LCS.

类似地，[5. 最长回文子串 ](https://leetcode.cn/problems/longest-palindromic-substring/)不就是最长公共子串嘛...还需要判断倒置前后的下标是不是匹配（排除原串本身有间隔的回文串）。

[139. 单词拆分](https://leetcode.cn/problems/word-break/)：这个就不是LCS了...

[72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

思路：你很难想到LCS，但是你可以理解为：删除即替换为空串，视增加为删除另一个串的字符，二者可以从不同视角看，进行转化。于是，当二者对应的字符不同时，修改的消耗是 `dp[i-1][j-1]+1`，插入 i 位置/删除 j 位置的消耗是 `dp[i][j-1] + 1`，插入 j 位置/删除 i 位置的消耗是 `dp[i-1][j] + 1`。

> 类似：[650. 只有两个键的键盘](https://leetcode-cn.com/problems/2-keys-keyboard/)便是乘法型的题。518也是。


##  背包/数组篇

[518. 零钱兑换 II](https://leetcode.cn/problems/coin-change-2/)



# 树专题

[226. 翻转二叉树 - 力扣（LeetCode）](https://leetcode.cn/problems/invert-binary-tree/):只能说这是梦开始的地方。这道题可以用前序、中序、后序和层次遍历解决，每种遍历都可以用递归和非递归实现。练手完毕后，欢迎进入树专题。

## 递归

[437. 路径总和 III - 力扣（LeetCode）](https://leetcode.cn/problems/path-sum-iii/)

思路：通过主函数、辅函数的协同，是解决树类问题的重要方法。

> 拓展：如何打印出所有路径？

[1110. 删点成林 - 力扣（LeetCode）](https://leetcode.cn/problems/delete-nodes-and-return-forest/)

类似前面正确地在递归方式下对链表断指针，此题对树进行处理。此外，可以通过哈希表查找是否是待删除节点，比多次在数组中`find`高效。如果需要练习树的指针处理，请通过就地方式解决[897.递增顺序查找树](https://leetcode.cn/problems/increasing-order-search-tree/solution/di-zeng-shun-xu-cha-zhao-shu-by-leetcode-dfrr/)

练习：[669. 修剪二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/trim-a-binary-search-tree/)
BST的基本查找、 结点增加与删除，以及AVL树的查找、结点增加与删除，也是递归便于解决的基本操作，可以参考[二叉搜索树简介 - OI Wiki ](https://oi-wiki.org/ds/bst/)学习。

## 综合

[99.恢复BST](https://leetcode-cn.com/problems/recover-binary-search-tree/)

提示：逆序对。拓展参考[剑指 Offer 51. 数组中的逆序对 ](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/submissions/)

[109. 有序链表转换二叉搜索树 - 力扣（LeetCode）](https://leetcode.cn/problems/convert-sorted-list-to-binary-search-tree/)

# 搜索专题



# 妖魔鬼怪专题

## 奇技淫巧系列

[318. 最大单词长度乘积](https://leetcode-cn.com/problems/maximum-product-of-word-lengths/)

为每个字母串建立一个长度为 26 的二进制数，每个位置表示是否存在该字母。同时，可以建立一个哈希表来存储字母串（在数组的位置）到二进制数字的映射关系。这样就能极快地判断判断两个字母串是否含有重复。

[260. 只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii/)

把所有的元素进行异或操作，得到结果`a^b`。但是为了得到`a` 和 `b` ，需要取异或值一个二进制位为 1 的数字作为 mask，分为每个数字都出现两次、有一个数字只出现了一次的两组，全异或就可以得到两个解。

> `a & (-a)` 可以获得a最低的非0位。

[769. 最多能完成排序的块](https://leetcode.cn/problems/max-chunks-to-make-sorted/)

注意到是**排列**，从左往右遍历，同时记录当前的最大值，每当当前最大值等于数组下标时，我们可以多一次分割。

[149. 直线上最多的点数](https://leetcode-cn.com/problems/max-points-on-a-line/)

枚举所有直线的过程不可避免，但统计点数的过程可以优化。对于每个点，我们对其它点建立哈希表，统计同一斜率的点一共有多少个。


## 设计数据结构系列

[LRU Cache - LeetCode](https://leetcode.com/problems/lru-cache/)

采用一个双向链表 list<pair<int, int>> 来储存信息的 key 和 value，维护链表的顺序是最新的信息在头节点。同时我们需要一个嵌套着链表的迭代器的 unordered_map<int, list<pair<int, int>>::iterator> 进行快速匹配（提示：存迭代器的原因是方便调用 `splice` 方法直接更新查找成功者到头部，当然也可以自己实现双向链表结构）。

**不建议使用`deque` 和 `vector` 的迭代器，会造成迭代器失效的问题**

练习（提示：哈希）：
[380. O(1) 时间插入、删除和获取随机元素 ](https://leetcode-cn.com/problems/insert-delete-getrandom-o1/)：充分利用数据结构的特性，比如说本题并不要求下标按序，那元素的排列特性能能否用于简化算法呢？
[460. LFU 缓存](https://leetcode.cn/problems/lfu-cache/)：著名LRU的pro版本。
[432. 全 O(1) 的数据结构](https://leetcode-cn.com/problems/all-oone-data-structure/)：LFU能否给你一点启发呢？
[859 · 最大栈 - LintCode](https://www.lintcode.com/problem/859/)：可以用类似 LRU 的方法降低时间复杂度

以上题的核心是哈希，下面是一些别的题目：

[208. 实现 Trie (前缀树) - 力扣（LeetCode）](https://leetcode.cn/problems/implement-trie-prefix-tree/)

有关更多数据结构方向的练习，欢迎参考Redis的笔记。【TBD】