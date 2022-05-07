-- 本章重点导航 -- 
[Leetcode-offer1](#JZ15%20位级运算)

> 环境配置：
>
> IDEA + LeetCode插件
>
> 插件的github上有相关issue，助你解决IDE的智能补全功能。

# Day 1

## JZ4：有序二维数组查找@二分思想

> 嘶，这个思想和汉明码神似！

==参：JZ11：二段有序数组最小值的二分查找法==

二分查找中，因为有序，`mid`实质是**判断在哪段，缩小解空间**。

而拓展二分思想，就是“分段思想”，从一个点的性质着手判断，位于哪段，缩小一段解空间。

```java
//[
//  [1,   4,  7, 11, 15],
//  [2,   5,  8, 12, 19],
//  [3,   6,  9, 16, 22],
//  [10, 13, 14, 17, 24],
//  [18, 21, 23, 26, 30]
//] 查找有无特定数字。
class Solution {
    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        int n = matrix.length; if(n == 0)  return false;
        int m = matrix[0].length; if(m == 0) return false;
        int tx = 0, ty = m - 1;
        while (matrix[tx][ty] != target){
            if( matrix[tx][ty] > target) ty--;
            else tx++;
            if(!inMatrix(tx,ty,n,m))    return false;
        }
        return true;
    }
}
```

你可以理解为，将矩阵旋转45度，得到类二叉搜索树；每次判断，剪枝一子树。

## JZ5：替换字符串空格为“%20”@StringBuilder

```java
//熟悉一下操作：
class Solution {
    public String replaceSpace(String s) {
        StringBuilder ans = new StringBuilder();
        for( Character tmp : s.toCharArray()){
            if(tmp == ' ')  ans.append("%20");
            else ans.append(tmp);
        }
        return ans.toString();
    }
}
```

## JZ7：由先序、中序遍历序列重建二叉树@HashMap

```java
//输入某二叉树的前序遍历和中序遍历的结果，请构建该二叉树并返回其根节点。 
// 假设输入的前序遍历和中序遍历的结果中都不含重复的数字。
// 示例: 
//
// 
//Input: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
//Output: [3,9,20,null,null,15,7]
```

本题主要学习了一种哈希优化递归方法，学习以下代码:

```java
class Solution {
    int[] preorder;
    HashMap<Integer, Integer> dic = new HashMap<>();
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        this.preorder = preorder;
        for(int i = 0; i < inorder.length; i++)
            dic.put(inorder[i], i);//由根节点值直接找到中序索引，免去二次查询
        return recur(0, 0, inorder.length - 1);
    }
    TreeNode recur(int root, int left, int right) {
        if(left > right) return null;                          
        TreeNode node = new TreeNode(preorder[root]);         
        int i = dic.get(preorder[root]);// 划分根节点、左子树、右子树
        node.left = recur(root + 1, left, i - 1);             
        node.right = recur(root + i - left + 1, i + 1, right); 
        return node;
    }
}
```

## 225：一个队列实现栈@Deque

用栈实现队列，需要两个栈，因为栈“LIFO”的特性，我们不能通过一个栈调转顺序。

而队列“LILO”的特性，使其“先出队后入队”后，能达到调转顺序的效果，因此，可以通过一个队列实现栈。

以下学习一下`Deque`的使用方式：

```java
Deque<String> deque = new LinkedList<>();
deque.offerLast("A"); // A
```

## 总结

遇有序，考虑二分（分段）！

# Day 2

## JZ12：矩阵路径@回溯法

解决一个回溯问题，实际上就是一个决策树的遍历过程，使用 `isValid` 函数剪枝。

回溯算法（DFS）模板框架如下：

```cpp
void backtracking(参数) {
    if (结束遍历条件) {
        存放结果;return;
    }
    for (选择：本层集合中元素（树中节点孩子的数量就是集合的大小）) {
        if (!isValid(..)) 	continue;//// 排除不合法选择
        进行回溯预备处理;
        backtracking(路径，选择列表); // 深入
        恢复现场；
    }
}
```

对于`isValid`函数及剪枝策略，回溯法求最值时，可以通过贪心思想找到一个“较优解”，减去差于其的枝。(分支限界法)

本题套模板不难，**注意入口**。

## JZ13：矩阵范围筛选@DFS/BFS

也可以使用回溯法解决的，学习一种操作： `return 1+dfs()+dfs()...`

**要理解回溯是为了什么，怎么准备现场，要不要恢复现场，怎么恢复现场**

BFS模板如下：

```java
int BFS(Node start, Node target) {
    Queue<Node> q;
    Set<Node> visited; // 避免走回头路，有替代方案
    q.offer(start); // 将起点加入队列
    visited.add(start);
    int step = 0;
    while (!q.isEmpty()) {//while:向下
        int sz = q.size();
        for (int i = 0; i < sz; i++) {//for：遍历该层
            Node cur = q.poll();
            //注意：此处可进行节点处理
            if (达成目标)     return step;
            for (Node x : cur.adj()) { //否则将cur的相邻节点加入队列
                if (!visited(x)) {
                    q.offer(x);
                    visited.add(x);
                }
            }
        }
        step++;//更新步数，在下一个元素出队之前
    }
}
```

# Day3

## JZ14 绳子分段积最大@动态规划 快速幂求余

模板：

动态规划实际是“聪明地枚举”，和回溯类似，会遍历所有可能的情况，但是我们会保存已经计算过的信息。其中，也允许“剪枝”（不丢失最优解）属于贪心的范畴，可能提高了“溯源”的难度。

首先注意：动态规划不是说一上来就写状态转移方程（直接做“备忘录”），而是逐步分析，如下：

分析最优**子结构** -> 递推公式 -> 自底向上求解（初始化、状态转移）

分析技巧： 递归树、状态转移图

调试技巧：打印 dp 数组

如果需要“溯源”，可以开一个新数组进行标记。

由于需要状态转移，一般数组下标从1开始，对应的`dp`数组也开有特别标识的“安全空间"。

注意状态转移的方向，利用了什么。

> C++中，可使用`max({a,b,c})`获得三个值的最值，Java似乎无类似机制。

**大数求余思想**

根据求余运算性质：$(xy) \% a = (x\%a \times y\%a)\%a$

结合二分快速幂  $x^a = x^{a/2} \times x^{a/2} \times x^{a -a/2\times2}$

即可达到快速幂求余的目的。

> 技巧：JAVA中的大数常量，可以使用“_”分隔，如：998_244_353

## JZ15 位级运算

《Hacker’s Delight》给予我们大量用位运算获取信息的技巧，整理部分如下：

```cpp
/*通过（x-1) (x+1)操作最右特定位
x - 1: 原最右的“1”变为“0”，最右边的“0”们变成“1”，
x + 1: 最右边的“1”们变成“0”，原最右的“0”变成“1”；0xFFFF->0x10000
*/
//使用&，意味着“0化”，改变原先的“1”，原先的“0”不变
x & (x - 1) //最右边的“1”置为“0”，值为0说明x只含一位“1”，为2的幂
x & (x + 1) //最右边的“1”们置为“0”, 值为0说明x类似于0b00..0011..11，即2^n - 1
//使用|，意味着“1化”，似上，原先的“1”不变
x | (x - 1) //最右边的“0”们置为“1”
x | (x + 1) //最右边的“0”置为“1”
//使用^，进行分化
x ^ (x - 1) //对最右边的“1”的右边“0”们都置“1”,其左边的位均置“0”
x ^ (x + 1) //对最右的“0”置为“1”,其左边的位均置“0”.

/* ~x：翻转位，运算符操作同上 */
~x & (x - 1) //最右边的“0”们变成“1”，其余位置“0”
~x & (x + 1) //最右边的“0”变成“1”，其余位置“0”
~x | (x - 1) //最右边的“1”变成“0”，并其余位置“1”
~x | (x + 1) //最右边的“1”们变成“0”，并将其余位置“1”
    
x & (-x) //获取最右侧位，因为 -x == ~x + 1
```

## JZ17：打印从1到最大的n位数

递归填充全排列。

# Day 4

## JZ19：字符串匹配

Java的`toCharArray`，并没有`arr[s.length] = ‘\0’ `，与访问器`charAt`一样，会抛出异常`ArrayIndexOutOfBoundsException`。

关键是，对两个串A和串B，**明确用谁去匹配谁**，把一个串当做另一个串的指针转移的条件，即“状态机”思想。

值得一提的是，本题可以使用动态规划的方式解决：

> 我们嫌参数名太长。既然是核心代码模式，你改短一点，也没什么所谓。

记`bool f[i][j]`代表 `s` 中的 `1~i` 字符和 `p `中的 `1~j` 字符是否匹配，留以`f[0][0]`为`true`进入空串匹配成功，后分为各种情况进行状态转移。其中，对于`p[j]` 为 `'*'`：读得 `p[j - 1]` 的字符， 可实际匹配 `s` 中 `a` 的个数是 0 个、1 个、2 个 ... 观察到（图源牛客题解）：

![640.png](http://img.070077.xyz/202202110344813.png)



## JZ21：数组划分

划分条件可变时，使用函数指针；

本题可使用双指针中的**快慢指针**技巧，是这样的：

```cpp
  int low = 0, fast = 0;
  while (fast < nums.size()) {
     if (nums[fast] & 1) {
          swap(nums[low], nums[fast]);
          low ++;
     }
     fast++;
  }
```

# Day5

接下来的题目就涉及到指针了，这个时候要记得

**访问元素前，记得特判`null`**

> 顺便可以复习一下JAVA基础中的`optional`类型。

遇到树的题目时，通用的思考方向：

**先考虑遍历一遍二叉树（所有结点），然后考虑分治成子问题进行递归。**

采用哪种遍历方式？只要明白：**已遍历结点向未遍历结点传递信息**即可。

回忆一下非递归中序遍历：

- 入栈顺序：先序遍历；
- 出栈顺序：中序遍历。

非递归后序遍历的实现：一种方法是，出栈时不马上弹出结点（处理的时机）。

> 复习： `poll` - 删除并返回 ；`peek` - 仅访问 

## JZ26：树的子结构

**分治**：以树为单位判断固然极难，以结点为单位判断就很简单了。

```java
public boolean isSubStructure(TreeNode A, TreeNode B) {
        if (B == null || A == null) return false;
        return Match(A, B) || isSubStructure(A.left, B) || isSubStructure(A.right, B);
    }
    private boolean Match(TreeNode A, TreeNode B){
        if (B == null)    return true;
        if (A == null || A.val != B.val)    return false;
        return Match(A.left,B.left) && Match(A.right,B.right);
    }
```

## JZ27：树的镜像

**如果调用的方法会改变树的结构，请检查调用参数是不是你所期待的！**

```java
mirrorTree(root.left);
//如果上面语句执行完毕后会改变root.right，那么下方语句的root.right就不是所希望的
mirrorTree(root.right);
```

一种补丁：

```java
TreeNode leftRoot = mirrorTree(root.right);
TreeNode rightRoot = mirrorTree(root.left);
root.left = leftRoot;   root.right = rightRoot;
```

# Day 6

## JZ29：顺时针打印矩阵@非方阵特殊处理

非方阵中，比较麻烦的是四角，在此提供两个特别用例：

$1\\2$  和 $1 \space\space 2$ .需要准确判断四角应该判作哪个角，以达到正确的目标。

## JZ30/31：魔改栈

任何的魔改需求，都可以加一层“辅助层”解决。

本题可以复习一下“常量池“，对于数字类包装，`[-128,127]`有缓存，可用`==`判等值；而其以外的*对象*，不可使用`==`。其实，对于对象，任何时候都应该优先考虑*使用方法*。

## JZ32：二叉树的花式遍历

本题不难，但是需要注意Java语法：

如果需要给一个对象的队列加入对象，**这个对象参数传入时是浅拷贝**，也就是说，**后续对对象本身的修改，会直接影响已入队的对象的值**。

提示：

1.类似`List<Integer> tmpq = new LinkedList<Integer>(tmp);`的操作，可以得到一个副本。

2.**` list.add(new LinkedList(alist))` ，可将 `alist` 的副本加入到 `list` 。**

```java
 public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> ans = new LinkedList<List<Integer>>();
        if( root == null) return ans;
        Queue<TreeNode> q = new LinkedList<TreeNode>();
        q.offer(root);
        while(!q.isEmpty()) {
            LinkedList<Integer> tmp = new LinkedList<>();
            for(int i = q.size(); i > 0; i--) {
                TreeNode node = q.poll();
                if(ans.size() % 2 == 0) tmp.addLast(node.val);
                else tmp.addFirst(node.val);
                if(node.left != null) q.add(node.left);
                if(node.right != null) q.add(node.right);
            }
            ans.add(tmp);
        }
        return ans;
    }
```

## JZ33：后序遍历序列判断@单调栈

**后序遍历倒序： [根|右|左] 。类似先序遍历的镜像**，对应大小关系：根 ↑右↓左。

这个单调性方便我们寻找“右与左的交界”：

借助一个单调栈 `stack` 存储值递增的节点；每当遇到值递减的节点 $r_i$，则通过出栈来更新节点 $r_i$ 的父节点 `root` （因为顺序是根在前）：这个时候处理完了一棵子树，结合根节点继续处理。

参考：[递归和栈两种方式解决，最好的击败了100%的用户 - 二叉搜索树的后序遍历序列 - 力扣（LeetCode） (leetcode-cn.com)](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/solution/di-gui-he-zhan-liang-chong-fang-shi-jie-jue-zui-ha/)

因为，对于逆序：

**挨着的两个数如果arr[i]<arr[i+1]，那么arr[i+1]一定是arr[i]的右子节点；如果arr[i]>arr[i+1]，那么arr[i+1]一定是arr[0]……arr[i]中某个节点的左子节点，并且这个值是大于arr[i+1]中最小的**。

# Day 7

## JZ34：二叉树和为某值的路径@回溯且溯源

DFS路径溯源需要设计一些序列化容器的方法使用，根据**多态**理论可以得到你想要的方法：

![集合继承关系图](http://img.070077.xyz/202202131818431.png)

> 本题可否使用BFS解决？
>
> 知道哪片叶子不难，怎么溯源路径呢：我们可以使用`map`对应结点的`parent`和路径和。
>
> 根据`parent`不断向上层即可，最后需要翻转。

## JZ35：复制盲盒链表@就地哈希

还记得吗，对于链表的题目，建议下笔画下来，再处理。

这里介绍一个神奇的思路：就地哈希。

由于本题中一个链表内有两个指针，进行复制时，很容易想到先按`next`指针复制一份链表（同时建立`HashMap`对应`random`指针），再通过哈希表进行新表的`random`连接。

我们有一个新想法：原地复制：A -> A’ -> B -> B’ -> …

一来沿着`next`直接得到了新链表，又可以非常方便地处理`random`，达到了就地哈希的效果。

## JZ36：BST双向链表化

中序遍历进行处理即可，关键是遍历的“第一个节点、最后一个节点”有无特性。

```java
Node pre, head;
    public Node treeToDoublyList(Node root) {
        if (root == null)   return null;
        dfs(root);
        head.left = pre;
        pre.right = head;
        return head;
    }
    private void dfs(Node root){
        if(root == null) return;
        dfs(root.left);
        if (pre != null) pre.right = root;
        else head = root;
        root.left = pre;
        pre = root;
        dfs(root.right);
    }
```



你可以借助JZ37，练习二叉树的非递归遍历及复现。

---

按输出顺序的倒序入栈，中节点入栈时**加入标志**NULL节点。例子：

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> result;
        stack<TreeNode*> st;
        if (root != NULL) st.push(root);
        while (!st.empty()) {
            TreeNode* node = st.top();
            if (node != NULL) {
                st.pop(); // 将该节点弹出，避免重复操作，下面再将右中左节点添加到栈中
                if (node->right) st.push(node->right);  // 添加右节点（空节点不入栈）

                st.push(node);                          // 添加中节点
                st.push(NULL); // 中节点访问过，但是还没有处理，加入空节点做为标记。

                if (node->left) st.push(node->left);    // 添加左节点（空节点不入栈）
            } else { // 只有遇到空节点的时候，才将下一个节点放进结果集
                st.pop();           // 将空节点弹出
                node = st.top();    // 重新取出栈中元素
                st.pop();
                result.push_back(node->val); // 加入到结果集
            }
        }
        return result;
    }
};
```
