---
title: "go 数据结构"
date: "2020-12-22"
tag: ["Golang"]
categories: ["Golang"]
draft: true
---

## 一.Slice

```go
// https://github.com/golang/go/blob/3b2a578166bdedd94110698c971ba8990771eb89/src/reflect/value.go#L1994
type SliceHeader struct {
  // 指向底层数组的指针
	Data uintptr 
  // 切片当前长度
	Len  int 
  // 切片当前容量
	Cap  int
}
```

### 1.1 Slice扩容策略

1. 如果期望容量>当前容量的两倍，使用期望容量
2. 切片长度<1024，容量翻倍 
3. 切片长度>1024，每次增加25%容量，直至新容量大于期望容量

> 大切片扩容或者复制时可能会发生大规模内存拷贝

## 二.哈希表

- 哈希函数
  - 决定哈希读写性能（键值映射）
- 冲突解决方法
  - 开放寻址法
    - **依次探测和比较数组中的元素以判断目标键值对是否存在于哈希表中**
    - 性能影响最大的是**装载因子 **（装载因子:=元素数量÷桶数量）
  - 拉链法
    - 引入红黑树以优化性能

```go
type hmap struct {
	count     int
	flags     uint8
	B         uint8
	noverflow uint16
	hash0     uint32

	buckets    unsafe.Pointer
	oldbuckets unsafe.Pointer
	nevacuate  uintptr

	extra *mapextra
}

type mapextra struct {
	overflow    *[]*bmap
	oldoverflow *[]*bmap
	nextOverflow *bmap
}
```

- 哈希扩容条件
  - 装载因子超过6.5
  - 哈希表使用太多溢出桶



## 三.字符串

> 只读的类型
> 做拼接和类型转换等操作时一定要注意性能的损耗

```go
type StringHeader struct {
	Data uintptr
	Len  int
}
```



