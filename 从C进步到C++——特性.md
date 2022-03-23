# 从C进步到C++——特性

@version 1.0

@update 20201005

[TOC]

## 结构体初始化构造

定义结构体时，加入以下语句，可以方便地进行初始化。
```c++
struct student{

	int id;
	char gender;
	student(){}  //系统默认，用于未经初始化定义结构体
	student (int _id, char _gender ){
	id = _id;
	gender = _gender;
	}//用于初始化id和gender。注意：减少参数 可以达到只使用部分元素构造的效果
}
```

构造函数可以简化为一行，并且可以进行默认赋值，如此调用时可以省略对应参数。如：

```cpp
Point (int x=0 , int y = 0):x(x),y(y) {};
//Point();
```



这是一个二叉树节点的定义示例：

```c++
struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};
```



## auto声明推断

使用STL容器时的迭代器声明等等往往比较复杂，此时使用 `auto` 可简单很多：

```c++
vector<int> vec;
auto pvec = vec.begin();//vector<int>::iterator
```



## 容器类for-range循环

对数组或容器执行循环操作可写成：

```c++
double prices[5] = {1.1 , 2.2 , 3.3 , 4.4 , 5.5}
for ( double x : prices){//只读
	cout<<x<<endl;
}
for ( double &x : prices){//写需要使用指针
	x = x / 2;
}
```



结合上面的auto 部分循环可以写成：

```
for ( auto x : vec )
```



## string类的类型转换函数

```cpp
int i = 43;
string s = to_string(i);
double d = stod(s);//43.000000
//对应的有：stoi , stoll , stof
```

**注意不要传入空串！**



## rotate/copy/fill函数

rotate函数可以“平移”。直接看例子吧：

```cpp
vector<int> a{1,2,3,4,5,6,7,8,9};
rotate(a.begin() , a.begin()+2 , a.end());
//a:[3,4,5,6,7,8,9,1,2]
```

相当于得到：[mid:end] + [begin:mid)



copy函数将一个目标（容器、数组）里面的元素复制至另一个目标。

**注：使用前 `newvector.resize(7) `这行代码为`newvector`分配空间，防止程序崩溃**。

此外，通过`vector<int>  dp ( 7 , 0 )`也可以达到初始化`dp`为[0,0,0,0,0,0,0].



fill函数可以为数组和vector赋初始值。头文件：`<algorithm>`

```c++
int v[10];
fill(v,v+10,-1);//相当于 memset(v,-1,sizeof v);
int s[10][10];
fill(s[0],s[0]+10*10,-1);
```

## sort/lower_bound/upper_bound/unique函数

sort函数可用于 `vector `、 数组 和 `deque` 的排序。如：

```cpp
sort(b,b+n,greater<int>());//将b降序排列
```

在**已排好升序**的情况下：

1.可使用lower_bound函数（`startptr`，`endptr` ，`num`）得到第一个不小于`num`的值的指针。（二分查找）。如下代码返回下标：

```cpp
int a[10];//......
int b = upper_bound(a , a+10 , 3) - a;
```

2.可使用unique函数（时间复杂度：O(n））返回最后一个非重复元素的指针，如下代码进行**去重**。

```cpp
vec.erase( unique(vec.begin(),vec.end()) , vec.end() );
```

拓展排序：partial_sort 方法



##  局部排序`partiai_sort`

可以提供一定区间的排序获得。原理似乎是堆排序。

```cpp
for(i=10;i>=1;i--)	vec.push_back(i);
partial_sort(vec.begin(),vec.begin()+3,vec.end());
//[1,2,3,10,9,8,7,6,5,4]

rep(i,1,10) a.push_back(i);
partial_sort(vec.begin(),vec.begin()+3,vec.end(),greater<int>());
//[10,9,8,1,2,3,4,5,6,7]
```

应用：班上有10个学生，我想知道分数最低的5名是哪些人。如果没有partial_sort，你就需要用sort把所有人排好序，然后再取前5个。现在你只需要对分数最低5名排序。

## 一些可能有用的数学函数

**指对运算**

以下函数：接收x为`float`	`double`类型，返回传入类型。

`exp2(x)`  - 快速返回2的x次幂。

`log2(x)` - 得到以2为底x的对数。

`log10(x)` - 得到以10为底x的对数。

`log(x)`  - 得到x的自然对数。

`expm(x)` - 返回e的x次幂-1。

`exp(x)` - 得到e的x次幂。x支持complex类型。

**取整函数**

以下函数的传入传出同上。

`floor(x)` - 向下取整。

`ceil(x)` - 向上取整。 

`round(x)` - 四舍五入，基本类似于`floor(x+0.5)`。**但是中点情况下向远离0的方向舍入**。

`trunc(x)` - 将x向0方向舍入。



## 数组生成- iota/shuffle

递增数组：与 `golang `可对比：

```cpp
iota(arr,arr+n,0);//[0,1,2,3,4,...];
```

随机打乱数组：

```cpp
srand(time(NULL));
random_shuffle(a+1,a+n+1);
```

`random_shuffle` 自 C++14 起被弃用，C++17 起被移除。可以使用 `shuffle` 函数代替：

`shuffle(v.begin(),v.end(),rand)`（最后一个参数传入的是使用的随机数生成器，一般情况下传入 `rand` 即可）。

使用时需要设置随机数种子，配合`iota`可以实现类似于“洗牌”的效果。同时可进行随机的多次模拟，为了时间种子的不同，可能需要每次模拟间设置`sleep`间隔。


[[C++STL的使用]]

参考资料：

《C++ Primer》 目录

《C++ Primer Plus》  18.1

《算法竞赛入门经典习题解答》第一章

http://www.cppblog.com/mzty/archive/2005/12/15/1770.aspx



