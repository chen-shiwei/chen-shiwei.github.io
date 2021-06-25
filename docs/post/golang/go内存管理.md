# 一般内存分配方法

- 线性分配器

  - 在内存中维护一个特定位置的指针，申请内存时分配根据指针位置检查可用剩余空闲内存、返回已分配的内存区域并修改指针在内存中的位置。没分配一次移动指针一次

    ![bump-allocator](https://img.draveness.me/2020-02-29-15829868066435-bump-allocator.png)

  - 缺点 已分配的内存释放后无法重用

    ![bump-allocator-reclaim-memory](https://img.draveness.me/2020-02-29-15829868066441-bump-allocator-reclaim-memory.png)

- 空闲链表分配器

  - 用链表的方式维护内存块，也解决了内存释放重用的问题

    ![free-list-allocator](https://img.draveness.me/2020-02-29-15829868066446-free-list-allocator.png)

## 空闲链表分配方式

- 首次适应	

  - 从链表头部开始遍历，找到第一个大于等于申请内存得内存块

- 循环首次适应

  - 从上一次遍历结束的位置开始遍历，找到第一个大于等于申请内存块

- 最优适应

  - 遍历链表找到最合适的内存块

- 隔离适应(go使用)

  - 将内存块切割为多个不同大小内存链表，每个链表的内存块大小一样，申请内存时找到合适大小的链表，再从链表找到合适的内存块

    ![segregated-list](https://img.draveness.me/2020-02-29-15829868066452-segregated-list.png)

go 内存分配策略

**使用多级缓存将对象根据大小分类，并按照类别实施不同的分配策略 **

TCMalloc（Thred-cache malloc）

## go 内存对象分类

- 微对象
  - 0-16b
- 小对象
  - 16b-32k
- 大对象
  - 32k+

## go 内存多级缓存

- 线程缓存
- 中心缓存
- 页堆

![multi-level-cache](https://img.draveness.me/2020-02-29-15829868066457-multi-level-cache.png)