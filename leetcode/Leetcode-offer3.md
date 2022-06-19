# 数学篇

# 数组篇

### 三数之和

[剑指 Offer II 007. 数组中和为 0 的三个数 - 力扣（LeetCode）](https://leetcode.cn/problems/1fGaJU/)

三指针。通过固定一个指针，变成双指针的两数之和问题。注意时时去重。

拓展：[16. 最接近的三数之和 - 力扣（LeetCode）](https://leetcode.cn/problems/3sum-closest/)：思路类似于上题，一样地需要注意去重

### 最值性质子数组

[剑指 Offer II 008. 和大于等于 target 的最短子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/2VG8Kg/)：子数组（连续）->滑动窗口；前缀和 + 二分查找也是一个不错的想法（记得最长递增子序列吗）

拓展：[718. 最长重复子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/maximum-length-of-repeated-subarray/)：滑动窗口和动态规划都是不错的思路。其中的滑动窗口有种KMP的意思。或许，本题其实就是最长公共子串..

[剑指 Offer II 011. 0 和 1 个数相同的子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/A1NYOS/)：技巧：把0看作-1

### 特定性质的子数组数

[剑指 Offer II 009. 乘积小于 K 的子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/ZVAVXX/)：本题求的是个数，需要简单变化一下...你可以拓展下思路：通过取对数**化乘为加**，避免整数溢出且便于二分查找。

[剑指 Offer II 010. 和为 k 的子数组 - 力扣（LeetCode）](https://leetcode.cn/problems/QTMn0o/)：前缀和 + 差分

[和为s的正数子数组](Leetcode-offer2.md)使用滑动窗口解决这一问题，而本题去除了这一限制，又要如何处理呢？


# 字符串篇

# 链表篇

# 哈希表篇

# 栈与队列

# 树

## 堆

## 前缀树

# 回溯法

# 动态规划

# 图
