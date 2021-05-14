---
title: "go slice深度分析"
date: "2021-02-20"
tag: ["Golang"]
categories: ["Golang","slice","type"]
draft: true
---

## slice 源代码
> https://golang.org/src/runtime/slice.go
```go
type slice struct {
   array unsafe.Pointer // 底层数组指针
   len   int // slice 长度
   cap   int // 容量 最大能够存的数量 cap >= len
}
```
![image](/img/slice.png)


```go
// 切片的声明
var nilSlice []int
newSlice := *new([]int)
slice := []int{}
makeSlice := make([]int, 0) // make 可以指定len和cap 第二个是len 第三个是cap
splitSlice := ([]int{})[:]
array := [...]int{}
splitArray := array[:]

fmt.Printf("var nilSlice []int p:%p len:%d cap:%d \n", nilSlice, len(nilSlice), cap(nilSlice))
fmt.Printf("newSlice := new([]int) p:%p len:%d cap:%d \n", newSlice, len(newSlice), cap(newSlice))
fmt.Printf("slice := []int{} p:%p len:%d cap:%d \n", slice, len(slice), cap(slice))
fmt.Printf("makeSlice := make([]int, 0) p:%p len:%d cap:%d \n", makeSlice, len(makeSlice), cap(makeSlice))
fmt.Printf("splitSlice := ([]int{1, 2})[:] p:%p len:%d cap:%d \n", splitSlice, len(splitSlice), cap(splitSlice))
fmt.Printf("array := [...]int{} \n splitArray := array[:] p:%p len:%d cap:%d \n", splitArray, len(splitArray), cap(splitArray))
// var nilSlice []int             // p:0x0 len:0 cap:0  空slice
// newSlice := new([]int)         // p:0x0 len:0 cap:0  空slice
// slice := []int{}               // p:0x119f428 len:0 cap:0 empty Slice
// makeSlice := make([]int, 0)    // p:0x119f428 len:0 cap:0  empty Slice
// splitSlice := ([]int{1, 2})[:] // p:0x119f428 len:0 cap:0 empty Slice
// array := [...]int{}
// splitArray := array[:] // p:0x119f428 len:0 cap:0 empty Slice

func() {
   var s1 []int
   fmt.Printf("s1 == nil %v \n", s1 == nil) // s1 == nil true
   s2 := []int{}
   fmt.Printf("s2 == nil %v \n", s2 == nil) //  s2 == nil false
   s3 := make([]int, 0)
   fmt.Printf("s3 == nil %v \n", s3 == nil) //  s3 == nil false
}()

// 指定索引9赋值10
var s1 = make([]int, 0)
fmt.Printf("s1:%v cap:%d len:%d\n", s1, cap(s1), len(s1))
// s1:[] cap:0 len:0
// 截取slice 创建新slice
sa := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
sb := sa[2:4:6] // [2,3]data[low, high, max] {low,...,height-1}
// sb := sa[:1:15] // panic: runtime error: slice bounds out of range [::15] with capacity 10
// 当 high == low 时，新 slice 为空。
fmt.Printf("sa %v :%p cap:%d len:%d\n", sa, sa, cap(sa), len(sa))
fmt.Printf("sb %v :%p cap:%d len:%d\n", sb, sb, cap(sb), len(sb))
// sa [0 1 2 3 4 5 6 7 8 9] :0xc0000ba000 cap:10 len:10
// sb [2 3] :0xc0000ba010 cap:4 len:2

// slice 扩容
var moreSlice = []int{0}
for i := 0; i < 4096; i++ {
   oldCap := cap(moreSlice)
   moreSlice = append(moreSlice, i)
   newCap := cap(moreSlice)
   if oldCap != newCap {
      fmt.Printf("growCap old:%d -> new:%d %2.f%%\n", oldCap, newCap, (float64(newCap)-float64(oldCap))/float64(oldCap)*100)
   }
}
// growCap old:1 -> new:2 100%
// growCap old:2 -> new:4 100%
// growCap old:4 -> new:8 100%
// growCap old:8 -> new:16 100%
// growCap old:16 -> new:32 100%
// growCap old:32 -> new:64 100%
// growCap old:64 -> new:128 100%
// growCap old:128 -> new:256 100%
// growCap old:256 -> new:512 100%
// growCap old:512 -> new:1024 100%
// growCap old:1024 -> new:1280 25%
// growCap old:1280 -> new:1696 32%
// growCap old:1696 -> new:2304 36%
// growCap old:2304 -> new:3072 33%
// growCap old:3072 -> new:4096 33%
// growCap old:4096 -> new:5120 25%

s := []int{1, 2}
s = append(s, 4)
s = append(s, 5)
s = append(s, 6)
fmt.Printf("len=%d, cap=%d", len(s), cap(s))
// len=5, cap=8%

a1 := []int{1, 2, 3}
a2 := a1
a2 = append(a2, 100)
a2[1] = 10

fmt.Println(a1, a2) // [1 2 3] [1 10 3 100] 扩容成新slice无影响a1
func() {
   // 浅拷贝和深拷贝
   s1 := make([]int, 5, 5)
   s2 := s1
   s1[1] = 1
   fmt.Printf("s1：%v ，s2：%v \n", s1, s2) // s1：[0 1 0 0 0] ，s2：[0 1 0 0 0]
   // 深拷贝
   s3 := make([]int, 5, 5)
   s4 := make([]int, 5, 5)
   copy(s4, s3)
   s3[1] = 1
   fmt.Printf("s3：%v ，s4：%v \n", s3, s4) // s3：[0 1 0 0 0] ，s4：[0 0 0 0 0]
}()
```

## FastHttp slice+sync.Pool 结构复用
> https://cbsheng.github.io/posts/fasthttp%E6%BA%90%E7%A0%81%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%E5%88%86%E6%9E%90/
## slice 内存泄露
> https://xargin.com/logic-of-slice-memory-leak/