# 数组/字符串系列

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
思路：为了方便计算天数差，我们这里存放下标而非温度本身，因为存放温度本身不便反算下标。

[456. 132 模式 ](https://leetcode-cn.com/problems/132-pattern/)



> 拓展问题：满足132模式的子序列有多少个。
> 甚至有一些有趣的研究。[Stack-sortable permutation - Wikipedia](https://en.wikipedia.org/wiki/Stack-sortable_permutation)



## 区间系列

[919 · 会议室 II - LintCode](https://www.lintcode.com/problem/919/description)

这道题可以用【前缀和】【扫描线】两种思路思考。（上面练过）

而且这个时候就不得不提一道经典题目了：

![](http://img.070077.xyz/202205070804953.png)


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

> 链表特别需要注意**考虑每个节点的指针**，拓展题：[25. K 个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)
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

[23. 合并K个升序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists/)

可以类似于归并排序进行两两合并，也可以把**所有的链表存储在一个优先队列**中，每次提取所有链表头部节点值最小的那个节点，直到所有链表都被提取完为止。


# 动态规划专题

## 字符串篇

> 万物皆可~~dp~~ LCS...

[516. 最长回文子序列](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

你应该记得......那是某个秋天，王晓茹的课上，循序渐进引出的例题......反转字符串+LCS.

[72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

思路：你很难想到LCS，但是你可以理解为：删除即替换为空串，视增加为删除另一个串的字符，二者可以从不同视角看，进行转化。于是，当二者对应的字符不同时，修改的消耗是 `dp[i-1][j-1]+1`，插入 i 位置/删除 j 位置的消耗是 `dp[i][j-1] + 1`，插入 j 位置/删除 i 位置的消耗是 `dp[i-1][j] + 1`。

> 类似：[650. 只有两个键的键盘](https://leetcode-cn.com/problems/2-keys-keyboard/)


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
