# 数据结构
## String

`<data []Byte, len int>`

#### UTF-8编码
![](http://img.070077.xyz/20221216204612.png)

## Slice

`<data []T, len int, cap int>`

#### 扩容规则
1. 预估扩容后容量
<img src="http://img.070077.xyz/20221216204925.png"/>
2. 分配内存，匹配合适的内存规格

## Map
#### 哈希表
![](http://img.070077.xyz/20221216205314.png)
（使用渐进式rehash进行扩容）

上图中右上角的结构即为`bmap`，上方是`topHash`值，接下来的分别是`[]keys, []values`

#### 扩容规则
![](http://img.070077.xyz/20221216205546.png)

# 闭包



---

参考资料：
[【幼麟实验室】Golang合辑_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1hv411x7we/?spm_id_from=333.999.0.0&vd_source=1e6ac4250e3c54ed91e0fe2b40f533ca)