# 1.数组
## 1.1概述
相同类型元素的集合组成的数据结构
```go
[2]int
[10]interface
```
数组大小在初始化之后无法改变
存储元素相同，并且大小也相同的不同数组才是同一类型

    a := [2]int{1, 2}
    b := [3]int{1, 2}
    fmt.Println(reflect.DeepEqual(a, b)) // false
## 1.2 初始化
```go
arr1 := [2]int{1,2}
// 上限推导，编译期推导数组大小
arr2 := [...]int{1,2}
fmt.Println(reflect.DeepEqual(arr1, arr2)) // true
```
根据元素数量不同，编译器在初始化字面量时做不同的优化

元素数量<=4，将元素中的元素放到栈上

大于 4将数组中的元素放到静态区并在运行时取出
## 1.3 访问
- 索引负数、越界、非整数 panic，直接使用整数或者常量访问数组会在编译期发现，若是使用变量代替索引去访问则无法发现

# 2.Slice （动态数组）
## 2.1概述
```go
[]int
[]interface
```
//https://github.com/golang/go/blob/3b2a578166bdedd94110698c971ba8990771eb89/src/reflect/value.go#L1994
type SliceHeader struct {
  // 指向底层数组的指针
    Data uintptr 
  // 切片当前长度
    Len  int 
  // 切片当前容量
    Cap  int
}
```
## 2.2 切片与数组的关系
切片引用抽象层
对数组中的部分连续片段进行引用
在运行期间可以修改它的长度和范围
当底层数组长度无法满足切片时会触发扩容，上层是无法看到切片变化
## 2.3 Slice容量不足时自动扩容策略
如果期望容量>当前容量的两倍，使用期望容量
切片长度<1024，容量翻倍
切片长度>1024，每次增加25%容量，直至新容量大于期望容量
大切片扩容或者复制时可能会发生大规模内存拷贝

golang-slice-append

## 2.4 切片初始化
​```go
arr[0:3] or slice[0:3]
slice := []int{1, 2, 3}
slice := make([]int, 10)
data[low, high, max] max: Invalid index values, must be low <= high <= max

    s := []int{1, 2, 3, 4, 5, 6, 7, 8}
    s1 := s[3:6:8]
    fmt.Println(s1, len(s1), cap(s1))
// [4 5 6] 3 5
```
## 2.5 访问
len 和 cap 获取长度或者容量是切片最常见的操作=>编译器将这它们看成两种特殊操作 OLEN 和 OCAP,len(slice) 或者 cap(slice) 在一些情况下会直接替换成切片的长度或者容量，不需要在运行时获取
# 3 哈希表
## 3.1 哈希函数
- 决定哈希读写性能（键值映射）
- 要求输出范围大于输入范围(理想的效果)
## 3.2 冲突解决方法
- 开放寻址法
- 依次探测和比较数组中的元素以判断目标键值对是否存在于哈希表中
- 性能影响最大的是装载因子 （装载因子:=元素数量÷桶数量）
  ![open-addressing-and-set](https://img.draveness.me/2019-12-30-15777168478785-open-addressing-and-set.png)
- 拉链法
 ![separate-chaing-and-set](https://img.draveness.me/2019-12-30-15777168478798-separate-chaing-and-set.png)
 - 引入红黑树以优化性能
## 3.3 数据结构
   ![hmap-and-buckets](https://img.draveness.me/2020-10-18-16030322432679/hmap-and-buckets.png)

```go
// 哈希结构，结合上图
type hmap struct {
  count     int // 哈希表的元素数量
  flags     uint8
  B         uint8 // 存bucket的数量对数，2的倍数 len(buckets) = 2^B
  noverflow uint16
  hash0     uint32 // 哈希的种子 给哈希结果引入随机性

  buckets    unsafe.Pointer // 指向 []*runtime.bmap,每个桶存8个键值对，单个桶装满使用extra.nextOverflow
  oldbuckets unsafe.Pointer // 扩容之前的buckets，大小是buckets的一半
  nevacuate  uintptr

  extra *mapextra
}

type mapextra struct {
  overflow    *[]*bmap
  oldoverflow *[]*bmap
  nextOverflow *bmap
}

// A bucket for a Go map. 编译
type bmap struct {
    // tophash generally contains the top byte of the hash value
    // for each key in this bucket. If tophash[0] < minTopHash,
    // tophash[0] is a bucket evacuation state instead.
    tophash [bucketCnt]uint8
    // Followed by bucketCnt keys and then bucketCnt elems.
    // NOTE: packing all the keys together and then all the elems together makes the
    // code a bit more complicated than alternating key/elem/key/elem/... but it allows
    // us to eliminate padding which would be needed for, e.g., map[int64]int8.
    // Followed by an overflow pointer.
}
// 运行时
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}

```
## 3.4 哈希扩容条件
- 装载因子超过6.5 （装载因子:=元素数量÷桶数量）
- 哈希表使用太多溢出桶

## 3.5 什么类型可以做哈希的
key Invalid map key type: the comparison operators == and != must be fully defined for key type

## 3.6 初始化
```go
func makemap(t *maptype, hint int, h *hmap) *hmap {
    mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
    if overflow || mem > maxAlloc {
        hint = 0
    }

    if h == nil {
        h = new(hmap)
    }
    h.hash0 = fastrand()
    
    B := uint8(0)
    for overLoadFactor(hint, B) {
        B++
    }
    h.B = B
    
    if h.B != 0 {
        var nextOverflow *bmap
        h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
        if nextOverflow != nil {
            h.extra = new(mapextra)
            h.extra.nextOverflow = nextOverflow
        }
    }
    return h
}
```
这个函数会按照下面的步骤执行： 
- 计算哈希占用的内存是否溢出或者超出能分配的最大值；
- 调用 runtime.fastrand 获取一个随机的哈希种子； 
- 根据传入的 hint 计算出需要的最小需要的桶的数量；
- 使用 runtime.makeBucketArray 创建用于保存桶的数组；

## 3.7 key 访问

![hashmap-mapaccess](https://img.draveness.me/2020-10-18-16030322432560/hashmap-mapaccess.png)
```go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    alg := t.key.alg
  // 计算哈希
    hash := alg.hash(key, uintptr(h.hash0))
  // 获取
    m := bucketMask(h.B)
  // 拿到hash到的桶
    b := (*bmap)(add(h.buckets, (hash&m)*uintptr(t.bucketsize)))
    top := tophash(hash)
bucketloop:
  // 在bmap和overflow 中找到对应的key
    for ; b != nil; b = b.overflow(t) {
        for i := uintptr(0); i < bucketCnt; i++ {
            if b.tophash[i] != top {
                if b.tophash[i] == emptyRest {
                    break bucketloop
                }
                continue
            }
            k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.keysize))
            if alg.equal(key, k) {
                v := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+i*uintptr(t.valuesize))
                return v
            }
        }
    }
    return unsafe.Pointer(&zeroVal[0])
}
```
## 3.8 扩容(渐进)
- 在扩容期间访问哈希表时会使用旧桶，向哈希表写入数据时会触发旧桶元素的分流。
- 除了这种正常的扩容之外，为了解决大量写入、删除造成的内存泄漏问题，哈希引入了 sameSizeGrow 这一机制，在出现较多溢出桶时会整理哈希的内存减少空间的占用。

- runtime: limit the number of map overflow buckets 引入了 sameSizeGrow 通过复用已有的哈希扩容机制解决该问题，一旦哈希中出现了过多的溢出桶，它会创建新桶保存数据，垃圾回收会清理老的溢出桶并释放内存5。

- 随着键值对数量的增加，溢出桶的数量和哈希的装载因子也会逐渐升高，超过一定范围就会触发扩容，扩容会将桶的数量翻倍，元素再分配的过程也是在调用写操作时增量进行的，不会造成性能的瞬时巨大抖动。

## 4.字符串
只读的类型s 做拼接和类型转换等操作时一定要注意性能的损耗

```go
type StringHeader struct {
    Data uintptr
    Len  int
}
```