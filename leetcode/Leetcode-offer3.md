# 数学篇

[剑指 Offer II 004. 只出现一次的数字（1/3） - 力扣（LeetCode）](https://leetcode.cn/problems/WGki4K/?favorite=e8X3pBZi)发现了人才的想法：与其在位运算这里摆弄状态机，不妨开个`32`长度的数组存储每一位出现的次数信息。

# 数组篇

### 三数之和

[剑指 Offer II 007. 数组中和为 0 的三个数 - 力扣（LeetCode）](https://leetcode.cn/problems/1fGaJU/)

三指针。通过固定一个指针，变成双指针的两数之和问题。注意时时去重。

拓展：[16. 最接近的三数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/3sum-closest/)：思路类似于上题，一样地需要注意去重. 

### 最值性质子数组

[剑指 Offer II 008. 和大于等于 target 的最短子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/2VG8Kg/)：子数组（连续）->滑动窗口；前缀和 + 二分查找也是一个不错的想法（记得最长递增子序列吗）

拓展：[718. 最长重复子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)：滑动窗口和动态规划都是不错的思路。其中的滑动窗口有种KMP的意思。或许，本题其实就是最长公共子串..

[剑指 Offer II 011. 0 和 1 个数相同的子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/A1NYOS/)：技巧：把0看作-1

### 特定性质的子数组数

[剑指 Offer II 009. 乘积小于 K 的子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/ZVAVXX/)：本题求的是个数，需要简单变化一下...你可以拓展下思路：通过取对数**化乘为加**，避免整数溢出且便于二分查找。**如果你想使用双指针，那么，
1. 每次右指针位移到一个新位置，应该加上 `r - l + 1` 种组合
2. 初始化的临时乘积非0，为1.

[剑指 Offer II 010. 和为 k 的子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/QTMn0o/)：前缀和 + 差分

[和为s的正数子数组](Leetcode-offer2.md)使用滑动窗口解决这一问题，而本题去除了这一限制，又要如何处理呢？

[剑指 Offer II 057. 值和下标之差都在给定的范围内 - 力扣（LeetCode）](https://leetcode.cn/problems/7WqeDu/) 使用滑动窗口 + 有序集合

# 字符串篇

[剑指 Offer II 020. 回文子字符串的个数 - 力扣（LeetCode）](https://leetcode.cn/problems/a7VOhD/?favorite=e8X3pBZi)：

枚举常有规律，此题如何DP？
```c++
for (int i = 0; i < len; i++) {
    ans++; dp[i][i] = true; // single char
    for (int j = 0; j < i; j++) {
        if (s[i] == s[j]) {
            if (j + 1 < i - 1 && !dp[j + 1][i - 1]) continue;
            dp[j][i] = true; ans++; // inner "" or Palindrome
        }
    }
}
```

实际上可以更好理解，中心拓展即可。（合并提示：`r = i % 2`）

By the way, 逆序也可以使用*栈* 思想。

拓展题：根据回文的性质，其实我们就是只需要根据末3个字母进行回文判断的。请见：[E. 不含回文子串的字符串数 - 北京邮电大学2022级校赛（预赛）](http://img.070077.xyz/20221029141544.png)及其[讲解笔记图](http://img.070077.xyz/VGQ`${X2U37(Y8TX6QHK$AQ.png)

# 链表篇

[剑指 Offer II 027. 回文链表 - 力扣（LeetCode）](https://leetcode.cn/problems/aMhZSa/)本题本身不难，但是[官方题解](https://leetcode.cn/problems/palindrome-linked-list/solution/hui-wen-lian-biao-by-leetcode-solution/)递归解法很有意思。
```c++
//如果使用递归反向迭代节点，同时使用递归函数**外**的变量向前迭代，就可以判断链表是否为回文。
class Solution {
    ListNode* frontPointer;
public:
    bool recursivelyCheck(ListNode* currentNode) {
        if (currentNode != nullptr) {
            if (!recursivelyCheck(currentNode->next)) {
                return false;
            } // 如果为真，说明在自顶向下递归调用（压栈），反之自底向上（弹栈）
            if (currentNode->val != frontPointer->val) {
                return false;
            } // 判断回文
            frontPointer = frontPointer->next;
        }
        return true; // 这里也是递归的base case
    }

    bool isPalindrome(ListNode* head) {
        frontPointer = head;
        return recursivelyCheck(head);
    }
};
```

# 哈希表篇

[剑指 Offer II 033. 变位词组 - 力扣（LeetCode）](https://leetcode.cn/problems/sfvd7V/)有用“字典”作为哈希表的`key`的，不过考虑到排列的性质，排序是一个比较妙的思路。

【设计数据结构系列】
[剑指 Offer II 030. 插入、删除和随机访问都是 O(1) 的容器 - 力扣（LeetCode）](https://leetcode.cn/problems/FortPu/?favorite=e8X3pBZi)

# 栈与队列

[剑指 Offer II 037. 小行星碰撞 - 力扣（LeetCode）](https://leetcode.cn/problems/XagZNi/)本题涉及末端元素的删除，而元素的下标并无意义。于是，我们利用栈模拟（可以就地模拟）。

【单调栈系列问题】

# 树

[剑指 Offer II 047. 二叉树剪枝 - 力扣（LeetCode）](https://leetcode.cn/problems/pOCWxh/?favorite=e8X3pBZi)

利用好递归函数的返回值：`root->left = pruneTree(root->left);`

[剑指 Offer II 050. 向下的路径节点之和 - 力扣（LeetCode）](https://leetcode.cn/problems/6eUYwP/?favorite=e8X3pBZi)
递归思路：进行遍历，每次遍历到节点时，作为`root`形成子问题（不改变`targetSum`），由辅助函数（改变`targetSum`）继续处理。

【树上前缀和】上面的想法显然存在较多的重复计算，我们只需要维护一个map，对应记录每个节点的路径和及此和出现的频率，即可差分得到结果。由于路径限制，在遍历完一个节点子树时，要将其路径和从map中移除，否则就是非法答案。

```c++
	prefix[curr]++;
	ret += dfs(root->left, curr, targetSum);
	ret += dfs(root->right, curr, targetSum);
	prefix[curr]--;
```

>如何输出路径？
>前缀和没有保留路径信息，回到传统的DFS.

[剑指 Offer II 054. 所有大于等于节点的值之和 - 力扣（LeetCode）](https://leetcode.cn/problems/w6cpku/)：解决树的问题的话，遍历是基本；BST，自然是中序遍历。本题关键是【逆中序遍历】！！！


## 堆

[剑指 Offer II 061. 和最小的 k 个数对 - 力扣（LeetCode）](https://leetcode.cn/problems/qn8gGX/?favorite=e8X3pBZi) 本题很容易想到利用最小堆，**但是如果太过无脑，会TLE。或者在重复元素的情况下摔跟头**。

引入一种新思想：【多指针归并】
因为两个数组均为有序的，因此每次取，只会基于**上次**选出的值，在该状态空间中增加两对。


## 前缀树

# 回溯法

# 动态规划

# 图
