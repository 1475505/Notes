***C++ STL的使用手册***

@version 1.1 

@updated 20220303

---


![img](https://oi-wiki.org/lang/csl/images/container1.png)

## 共有函数

`=`：有赋值运算符以及复制构造函数。

`begin()`：返回指向开头元素的迭代器。

`end()`：返回指向末尾的下一个元素的迭代器。`end()` **不指向某个元素**，它是末尾元素的后继。

`size()`：返回容器内的元素个数。

`max_size()`：返回容器 **理论上** 能存储的最大元素个数。依容器类型和所存储变量的类型而变。

`empty()`：返回容器是否为空的一个 `bool` 值，即 `begin() == end()`，`true` 为空，`false` 为非空。

`swap()`：交换两个容器。

`clear()`：清空容器。

`==`/`!=`/`<`/`>`/`<=`/`>=`：按 **字典序** 比较两个容器的大小。（比较元素大小时 `map` 的每个元素相当于 `set<pair<key, value> >`，`pair` 按 first 到 second 的顺序比较。无序容器不支持 `<`/`>`/`<=`/`>=`）

## 迭代器

迭代器（Iterator）可以看作一个*数据指针*，用来访问和检查 STL 容器中元素的对象。主要支持两个运算符：

![](http://img.070077.xyz/202205080421883.png)

-   `vector` `deque`：随机访问迭代器，支持`++` `--` `+n` `>` `[]`
-   `list` `set`  `map`：双向迭代器，支持`++` `--`

# 序列式容器

## vector

创建容器：如`vector<int> dp(4, 1)`可初始化`dp`为[1, 1, 1, 1].

**元素访问**

1. `at()`

   `v.at(pos)` 返回容器中下标为 `pos` 的引用。如果数组越界抛出 `std::out_of_range` 类型的异常。

2. `operator[]`

   `v[pos]` 返回容器中下标为 `pos` 的引用。不执行越界检查。

3. `front()`

   `v.front()` 返回首元素的引用。

4. `back()`

   `v.back()` 返回末尾元素的引用。
      
 可以认为，`v[i]`与`*(v.begin()+i)`等价。但，**除`vector` `string`外**，其他STL容器不支持后者的形式。

- `size()` 返回容器长度（元素数量），即 `std::distance(v.begin(), v.end())`。

- `insert()` 支持在某个迭代器位置插入元素、可以插入多个。**复杂度与 `pos` 距离末尾长度成线性而非常数**

- `erase()` 删除某个**迭代器或者区间的元素**，返回最后被删除的迭代器。复杂度与 `insert` 一致。

- `push_back(x)` 在末尾插入一个元素x，均摊复杂度：常数，最坏为线性复杂度。

	> `push_back` 会先创建元素x，再将这个元素拷贝或者移动到容器尾部（拷贝后会自行析构先前创建的这个元素）
	> `emplace_back()` *（C++11）* 功能与上一相同。实现时，直接在容器尾部创建元素x，省去了拷贝或移动元素的过程。

- `pop_back()` 删除末尾元素，常数复杂度。

## deque

一个非常强大的数据结构，既支持 O(1) *随机*读取，又支持 O(1) 的头部增删和尾部增删，不过有一定的额外开销。操作大同`vector`。（可认为底层是连续的）

![](http://img.070077.xyz/202205011714469.png)


- `push_front()` 在头部插入一个元素。
- `pop_front()` 删除头部元素。

- `push_back()` 在末尾插入一个元素。
- `pop_back()` 删除末尾元素。

- `front()` 返回首元素的引用。
- `back()` 返回末尾元素的引用。

- 任意位置插入一个元素：`deq.insert(iterator it, const T& x);`
- 任意位置删除一个元素：`deq.erase(iterator it);`

## list

**双向链表**，与`deque`大致相同，但是由于 `list` 的实现是链表，因此不提供随机访问的接口。若需要访问中间元素，则需要使用迭代器。相较于 deque ，如果需要大量的插入和删除，而不关心随机存取，则应使用list。

## array(C++11)

| 成员函数    | 作用                                      | 示例         |
| :---------- | :---------------------------------------- | ------------ |
| `operator=` | `array2` 的每个元素重写 `array1` 对应元素 |              |
| `max_size`  | 返回可容纳的最大元素数                    |              |
| `fill`      | 以指定值填充容器                          | arr.fill(1); |
| `swap`      | 交换，交换array的复杂度为：O(size)        |              |

| 非成员函数   | 作用                          |
| :----------- | :---------------------------- |
| `operator==` | 按照字典序比较 `array` 中的值 |
| `std::get`   | 访问 `array` 的一个元素       |

# 关联式容器

## 统一共有函数

- `find(x)`: 若容器内存在**键**为 x 的元素，会返回该元素的**迭代器**；否则返回 `end()`。

  ![](http://img.070077.xyz/202202121720321.png)

  如何获取值呢？使用访问方法。

- `erase(pos)` 删除迭代器为 pos 的元素，可结合`find`函数使用。

- `erase(first,last)` 删除迭代器在 [first,last)范围内的所有元素。

- `count(x)`: 返回容器内键为 x 的元素**数量**。

- `lower_bound(x)`: 返回指向首个**不小于** 给定**键**的元素的迭代器。

- `upper_bound(x)`: 返回指向首个**大于 **给定键的元素的迭代器。若容器内所有元素均小于或等于给定键，返回 `end()`。

- `size()`: 返回容器内元素个数。


## set

`set` 内部采用红黑树实现。平衡二叉树的特性使得 `set` 非常适合处理需要同时兼顾查找、插入与删除的情况。

- `insert(x)` 将元素 x 插入到 `set` 中。自动排序和去重。
- `erase(x)` 删除值为 x 的 **所有** 元素，返回删除元素的个数。
- `erase(first,last)` 删除迭代器在 [first,last)范围内的所有元素。

`set` 在默认情况下的比较函数为 `<`（如果是非内置类型需要 [重载 `<` 运算符](https://oi-wiki.org/lang/op-overload/#compare)）。然而在某些特殊情况下，我们希望能自定义 `set` 内部的比较方式。这时候可以通过传入自定义比较器来解决问题。

具体来说，我们需要定义一个类，并在这个类中重写仿函数 [重载 `()` 运算符](https://oi-wiki.org/lang/op-overload/#function)。

```c++
struct cmp {  
	bool operator()(int a, int b) { return a > b; } //小根堆
};  
set<int, cmp> s;

//2022年了，或许可以使用lambda表达式？
```

## map

`map` 重载了 `operator[]`，可以用任意定义了 `operator <` 的类型作为下标（在 `map` 中叫做 `key`，也就是索引）。

- 通过向 `map` 中插入一个类型为 `pair<Key, T>` 的值可以达到插入元素的目的，例如 `mp.insert(pair<string,int>("Alan",100));`
- 可以直接通过下标访问来进行查询或插入操作。例： `mp["Alan"]=100`。利用下标访问 时，如果 `map` 中不存在相应键的元素，会自动在 `map` 中插入一个新元素，并将其值设置为默认值（对于整数，值为零；对于有默认构造函数的类型，会调用默认构造函数进行初始化）
- `erase(key)` 函数会删除**键**为 `key` 的 **所有** 元素。返回值为删除元素的数量。

## 无序关联式/哈希适配容器

四种基于哈希实现的无序关联式容器：`unordered_set`，`unordered_multiset`，`unordered_map`，`unordered_multimap`。

其操作与关联式容器类似。其`count`平均时间复杂度是O(1)，优势在我

### unordered_map

常用于作为哈希表的模板，其对`[]`的重载赋予了它数组般的特性存在。

### unordered_set

基于哈希表实现的【桶】。

> set底层是红黑树，count时间复杂度就是O(logN)。

# 容器适配器

为什么称为容器适配器呢，是因为其内部的实现是基于`vector`  `deque ` 等等，其原理可以参考《STL源码剖析》。

## 后进先出的stack

仅支持查询或删除最后一个加入的元素（栈顶元素），不支持随机访问。

- `top()` 访问栈顶元素（如果栈为空，此处会出错）
- `push(x)` 向栈中插入元素 x
- `pop()` 删除栈顶元素

## 先进先出的queue

- `front()` 访问队首元素（如果队列为空，此处会出错）
- `push(x)` 向队列中插入元素 x
- `pop()` 删除队首元素

## 优先队列

- `top()` 访问堆顶元素（此时优先队列不能为空）

- `push(x)` 插入元素，并对底层容器排序

- `pop()` 删除堆顶元素（此时优先队列不能为空）

以上`pop`方法返回`void`，不返回已删除元素的值。

- 优先级的定义：

  对于数字，一般是值大者优先。即默认为（大根堆）：

  `priority_queue<int, vector<int>, less<int> > pq;`

  第二个参数是内部实现方式；第三个参数 **less表示数字大的优先级越大。如果是`greater<int>`，则为数字小者优先级大。** 优先级大者，位于`top`。

  可以通过类似于关联式容器的方式定义优先级，如最大堆。参见`set`章节。

## pair的常见用法

头文件：`<utility>`

可当作（first，second…）两个元素的结构体，按照正常的结构体访问。

> `pair`的默认比较方式是：先比较`first`，再比较`second`。

## string的常见用法

`string`的实现形式类似于字符型`vector`，支持`push_back`、直接赋值（注意使用**双引号**）和下标访问。只支持`cin` `cout`，不支持`printf`等。

- 可以通过加减运算符实现字符串拼接和删减、比较运算符进行字典序比较。

- `substr(pos,len)`方法截取子串。时间复杂度是O（len）。

- `find(x,pos)`指定了开始寻找的位置为下标`pos`

- 上面的共有函数（`erase` `find`等）也是支持的。复杂度都是O(n)

  ```cpp
  //一个插入字符串的示例
  string s1 = "Hello";
  string s2 = "world";
  s1.insert(3,s2);//Helworldlo
  ```
  

# STL算法

在《特性》篇也介绍有一些用法。

- `find_end`：逆序查找。`find_end(v.begin(), v.end(), value)`。
- `nth_element`：按指定范围进行分类，即找出序列中第 *n* 大的元素，使其左边均为小于它的数，右边均为大于它的数。`nth_element(v.begin(), v.begin() + mid, v.end(), cmp)` 或 `nth_element(a + begin, a + begin + mid, a + end, cmp)`
- `next_permutation`：将当前排列更改为 **全排列中的下一个排列**。如果当前排列已经是 **全排列中的最后一个排列**（元素完全从大到小排列），函数返回 `false` 并将排列更改为 **全排列中的第一个排列**（元素完全从小到大排列）；否则，函数返回 `true`。`next_permutation(v.begin(), v.end())` 或 `next_permutation(v + begin, v + end)`。算法实现：[Leetcode-offer2](Leetcode-offer2.md)
- `reverse(vec.begin(), vec.end())`可以获得序列式容器的反转。

## bitset

通过固定的优化，使得一个字节的八个比特能分别储存 8 位的 `0/1`。

- `bitset()`: 初始化每一位都是 `false`。
- `bitset(int val)`: 设为 `val` 的二进制形式。
- `bitset(const string& str)`: 设为 01 串 `str`。

```cpp
bitset<4> bitset1;　　//无初始化下，默认每一位均为0
bitset<8> bitset2(16);　//保存长度为8的16的二进制表示：[00010000]
string s = "10010";
bitset<8> bitset3(s);　　//长度为10，前补0。[00010010]
```

- `operator []`: 访问其特定的一位。
- `operator ==/!=`: 比较两个 `bitset` 内容是否完全一样。
- `operator &/&=/|/| =/^/^=/~`: 进行按位与/或/异或/取反操作。**`bitset` 只能与 `bitset` 进行位运算**，若要和整型进行位运算，要先将整型转换为 `bitset`。
- `operator <>/<<=/>>=`: 进行二进制左移/右移。
- `operator <>`: 流运算符，这意味着你可以通过 `cin/cout` 进行输入输出。
- `count()`: 返回 `true` 的数量。
- `size()`: 返回 `bitset` 的大小。
- `_Find_first()`: 返回 `bitset` 第一个 `true` 的下标，若没有 `true` 则返回 `bitset` 的大小。
- `_Find_next(pos)`: 返回 `pos` 后面（下标严格大于 `pos` 的位置）第一个 `true` 的下标，若 `pos` 后面没有 `true` 则返回 `bitset` 的大小。
- `any()`: 若存在某一位是 `true` 则返回 `true`，否则返回 `false`。
- `none()`: 若所有位都是 `false` 则返回 `true`，否则返回 `false`。
- `all()`:*C++11*，若所有位都是 `true` 则返回 `true`，否则返回 `false`。

以下三个函数，若不传入参，对**所有bit**都执行对应的操作。
- `set(pos, val = true)`: 将某一位设置成 `true`/`false`。
- `reset(pos)`: 将某一位设置成 `false`。相当于 `set(pos, false)`。
-  `flip(pos)`: 翻转某一位。
  
---

参考资料：

OI-wiki

《算法笔记》