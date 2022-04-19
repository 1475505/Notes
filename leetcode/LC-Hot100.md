## 数组/字符串系列

双指针是在时空复杂度限制下，比较理想的思考方案，滑动窗口、快速选取都可以归到这一领域。

### 滑动窗口

[76. 最小覆盖子串 ](https://leetcode-cn.com/problems/minimum-window-substring/)

思路：使用哈希表计数索引，用哈希表动态维护窗口中所有字符及个数。先不断`j++`扩大窗口，使得窗口内包含T的所有元素，再`i++`缩小窗口，记录下这一流程下的必要最小值后，放弃这个位置，重复上述过程。

### 选取系列

荷兰国旗问题、topK问题在《剑指Offer》笔记中已提及，此处略。

[524. 通过删除字母匹配到字典里最长单词](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)
思路：归并两个有序数组。通过双指针匹配“相邻”时，需要特别注意。


## 链表专题

### 反转链表

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


### 环形链表

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

### 链表归并排序
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

## 动态规划专题

### 字符串篇

[72. 编辑距离](https://leetcode-cn.com/problems/edit-distance/)

思路：你很难想到LCS，但是你可以理解为：删除即替换为空串，视增加为删除另一个串的字符，二者可以从不同视角看，进行转化。于是，当二者对应的字符不同时，修改的消耗是 `dp[i-1][j-1]+1`，插入 i 位置/删除 j 位置的消耗是 `dp[i][j-1] + 1`，插入 j 位置/删除 i 位置的消耗是 `dp[i-1][j] + 1`。

> 类似：[650. 只有两个键的键盘](https://leetcode-cn.com/problems/2-keys-keyboard/)

