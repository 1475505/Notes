# 数据结构
> 通用规则：
> - create if not exists：如果容器不存在，那就创建一个，再进行操作。
> - drop if no elements：如果容器里的元素没有了，那么立即删除容器，释放内存。
> - 可以以对象为单位，设置过期时间。
## String

## list
可以理解为双向链表，常用来做异步队列使用，一般不用作栈。
`lpush lpop rpush rpop`

#### 底层实现
实际上是“快速链表”（quicklist）结构。首先在列表元素较少的情况下，会使用一块连续的内存存储，这个结构是ziplist（压缩列表）。它将所有的元素彼此紧挨着一起存储，分配的是－块连续的内存。当数据量比较多的时候才会改成 quicklist。也就是将多个 ziplist 使用双向指针串起来使用。

## hash
无序字典。Redis 的字典的值只能是字符串。
#### 渐进式rehash
在 rehash 的同时，保留新旧两个 hash 结构。

## set
Set 类型的底层数据结构是由哈希表或整数集合实现的，支持并交差等操作。

## zset
有序列表。内部实现类似于跳表。
`zadd zrange zrevrange zcard(count) zrem(remove)`








---
参考：
《Redis深度历险》
https://xiaolincoding.com/redis
