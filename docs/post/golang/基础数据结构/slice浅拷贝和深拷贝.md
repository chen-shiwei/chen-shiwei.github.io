---
title: "深拷贝和浅拷贝"
date: "2020-12-03"
tag: ["Golang"]
categories: ["Golang"]
draft: true
---


# slice 深拷贝和浅拷贝

## go slice 底层是基于数组实现的

```go
var array [2]int //数组
var slice []int // 切片
```

## 浅拷贝

源切片和目的切片共享同一底层数组

```go
s1 := make([]int, 5, 5)
s2 := s1
s1[1] = 1
fmt.Printf("s1：%v ，s2：%v \n", s1, s2) // s1：[0 1 0 0 0] ，s2：[0 1 0 0 0]
```

## 深拷贝

s3 源切片修改不影响 s4

```go
s3 := make([]int, 5, 5)
s4 := make([]int, 5, 5)
copy(s4, s3)
s3[1] = 1
fmt.Printf("s3：%v ，s4：%v \n", s3, s4) // s3：[0 1 0 0 0] ，s4：[0 0 0 0 0]
```

超出底层数组容量，会导致也会切片深拷贝

```go
s5 := []int{1, 2, 3, 4, 5}
s6 := s5
s5 = append(s5, 6)
fmt.Printf("s5：%v ，s6：%v \n", s5, s6) // s5：[1 1 3 4 5 6] ，s6：[1 1 3 4 5]
s5[2] = 0
fmt.Printf("s5：%v ，s6：%v \n", s5, s6) // s5：[1 2 0 4 5 6] ，s6：[1 2 3 4 5]
s6[2] = 8
fmt.Printf("s5：%v ，s6：%v \n", s5, s6) // s5：[1 2 0 4 5 6] ，s6：[1 2 8 4 5]
```
