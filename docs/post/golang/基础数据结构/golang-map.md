---
title: "go map"
date: "2021-02-22"
tag: ["Golang","map"]
categories: ["Golang"]
draft: false
---
## 1. 什么是map？
- 关联数组 
- map
- 字典
- 符号表
> kv组成的抽象数据结构，key具有唯一性，支持增删改查，平均查找效率为O(1)，最差为O(n)。

## 2. 实现方式
- 哈希查
    - 利用hash func 分配key到bucket，主要开销在hashfunc和数组常数访问
    - 哈希表碰撞
        - 拉链法
            - 将bucket实现为链表，碰撞新增在链表后插入
        - 链表法
            - 碰撞在buckets数组后增加空位
- 搜索树
    - AVL树
    - 红黑树

> 搜索树实现的key 遍历是有序的，而哈希表示无序的。


## 3. Go 的map怎么实现的？
> 哈希查找表+链表法
 代码结构
```go
// A header for a Go map.
type hmap struct {
    // 元素个数，调用 len(map) 时，直接返回此值
    count     int
    flags     uint8
    // buckets 的对数 log_2
    B         uint8
    // overflow 的 bucket 近似数
    noverflow uint16
    // 计算 key 的哈希的时候会传入哈希函数
    hash0     uint32
    // 指向 buckets 数组，大小为 2^B
    // 如果元素个数为0，就为 nil
    buckets    unsafe.Pointer
    // 扩容的时候，buckets 长度会是 oldbuckets 的两倍
    oldbuckets unsafe.Pointer
    // 指示扩容进度，小于此地址的 buckets 迁移完成
    nevacuate  uintptr
    extra *mapextra // optional fields
}

// mapextra holds fields that are not present on all maps.
type mapextra struct {
    // If both key and elem do not contain pointers and are inline, then we mark bucket
    // type as containing no pointers. This avoids scanning such maps.
    // However, bmap.overflow is a pointer. In order to keep overflow buckets
    // alive, we store pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
    // overflow and oldoverflow are only used if key and elem do not contain pointers.
    // overflow contains overflow buckets for hmap.buckets.
    // oldoverflow contains overflow buckets for hmap.oldbuckets.
    // The indirection allows to store a pointer to the slice in hiter.
    overflow    *[]*bmap
    oldoverflow *[]*bmap
    
    // nextOverflow holds a pointer to a free overflow bucket.
    nextOverflow *bmap
}

// A bucket for a Go map.
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
```

## 4.存放key流程/如何定位key的所在位置？
```go
10010111 | 000011110110110010001111001010100010010110010101010 │ 01010
// 最后B位为10 放在第10个bucket
// 用hash的前8位确定放在桶中([8]keys,[8]values)的哪个位置
```
1. 计算key hash
2. 用hash的最后B位确定存在在哪个桶([]bmap)
3. 用hash的前8位确定放在桶中([8]keys,[8]values)的哪个位置
## 5. map 在当做函数参数时，在函数内修改影响map
```go
// makemap 返回是指针
func makemap(t *maptype, hint int, h *hmap) *hmap
```
## 6. Map 两种get方式
```go
var m = map[string]int{"a":0,"b":2}
// 第一种
fmt.Println(m["a"]) // 0
// 第二种
a,ok := m["a"]
fmt.Println(a,ok) // 0 true
```
```go
// 底层函数实现
// src/runtime/hashmap.go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)
```
## 7. 扩容
> 装载因子为6.5
> 
> 扩容是渐进式，防止一次搬迁key过多影响性能
> 
> 触发扩容的时机: 新增新元素
> 
> bucket搬迁的触发: 赋值、删除时，每次搬迁两个bucket
```go
// src/runtime/map.go
// Maximum average load of a bucket that triggers growth is 6.5.
// Represent as loadFactorNum/loadFactorDen, to allow integer math.
loadFactorNum = 13
loadFactorDen = 2

loadFactor := count / (2^B)
// 扩容条件

// Did not find mapping for key. Allocate new cell & add entry.

// If we hit the max load factor or we have too many overflow buckets,
// and we're not already in the middle of growing, start growing.
// 触发最大负载因子或者太多的溢出桶
if !h.growing()/*没有正在扩容*/ && (overLoadFactor(h.count+1, h.B)/*判断负载因子*/ || tooManyOverflowBuckets(h.noverflow, h.B)) {
    hashGrow(t, h)
    goto again // Growing the table invalidates everything, so try again
}
// 负载因子计算
// overLoadFactor reports whether count items placed in 1<<B buckets is over loadFactor.
func overLoadFactor(count int, B uint8) bool {
    return count > bucketCnt && uintptr(count) > loadFactorNum*(bucketShift(B)/loadFactorDen)
}
// 溢出桶
// tooManyOverflowBuckets reports whether noverflow buckets is too many for a map with 1<<B buckets.
// Note that most of these overflow buckets must be in sparse use;
// if use was dense, then we'd have already triggered regular map growth.
func tooManyOverflowBuckets(noverflow uint16, B uint8) bool {
// If the threshold is too low, we do extraneous work.
// If the threshold is too high, maps that grow and shrink can hold on to lots of unused memory.
// "too many" means (approximately) as many overflow buckets as regular buckets.
// See incrnoverflow for more details.
// B的值大于15
if B > 15 {
B = 15
}
// The compiler doesn't see here that B < 16; mask B to generate shorter shift code.
return noverflow >= uint16(1)<<(B&15)
}
// 执行扩容
func hashGrow(t *maptype, h *hmap) {
    // If we've hit the load factor, get bigger.
    // Otherwise, there are too many overflow buckets,
    // so keep the same number of buckets and "grow" laterally.
    bigger := uint8(1)
    if !overLoadFactor(h.count+1, h.B) {
        bigger = 0
        h.flags |= sameSizeGrow
    }
    // 将buckets 搬迁 oldbuckets
    oldbuckets := h.buckets
    // 创建新的buckets
    newbuckets, nextOverflow := makeBucketArray(t, h.B+bigger, nil)
    
    flags := h.flags &^ (iterator | oldIterator)
    if h.flags&iterator != 0 {
        flags |= oldIterator
    }
    // commit the grow (atomic wrt gc)
    h.B += bigger
    h.flags = flags
    h.oldbuckets = oldbuckets
    h.buckets = newbuckets
    // 扩容进度
    h.nevacuate = 0
    h.noverflow = 0
    
    if h.extra != nil && h.extra.overflow != nil {
        // Promote current overflow buckets to the old generation.
        if h.extra.oldoverflow != nil {
            throw("oldoverflow is not nil")
        }
        h.extra.oldoverflow = h.extra.overflow
        h.extra.overflow = nil
    }
    if nextOverflow != nil {
        if h.extra == nil {
            h.extra = new(mapextra)
        }
        h.extra.nextOverflow = nextOverflow
    }

    // the actual copying of the hash table data is done incrementally
    // growWork 是渐进式执行搬迁的函数
    // by growWork() and evacuate().
}

func growWork(t *maptype, h *hmap, bucket uintptr) {
    // make sure we evacuate the oldbucket corresponding
    // to the bucket we're about to use
    evacuate(t, h, bucket&h.oldbucketmask())
    
    // evacuate one more oldbucket to make progress on growing
    // 判断是否搬迁完
    if h.growing() {
        evacuate(t, h, h.nevacuate)
        }
    }
}
```

## 8.map 比较
通过遍历比较key、val，无法对比(map1==map2 会无法编译)。