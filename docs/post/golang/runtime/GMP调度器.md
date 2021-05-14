---
title: "GMP调度器"
date: "2020-12-15"
tag: ["Golang","goroutine","GMP"]
categories: ["Golang"]
draft: true
---

# go sheduler
## 调度方式
采用协作式调度（由用户设置调度点称为协作式调度），但是也会有后台线程持续监控，超过10ms，会设置goroutine抢占标志位，调度器会处理。
```go
// scheduler 的陷阱
// 启动和机器的 CPU 核心数相等的 goroutine，每个 goroutine 都会执行一个无限循环。
// 创建完这些 goroutines 后，main 函数里执行一条 time.Sleep(time.Second) 语句。Go scheduler 因为超过10ms把主 goroutine 被调度走。先前创建的 threads 个 goroutines，刚好把 M 和 P 都占满了。
// 在这些 goroutine 内部，没有调用一些诸如 channel，time.sleep 这些会引发调度器工作的事情。只能任由这些无限循环执行下去了
// main goroutine 没有机会调度执行
func main() {
    var x int
    threads := runtime.GOMAXPROCS(0)
    for i := 0; i < threads; i++ {
        go func() {
            for { x++ }
        }()
    }
    time.Sleep(time.Second)
    fmt.Println("x =", x)
}
```
## go程序执行组成
- GoProgram(用户代码程序)
- Runtime(运行时)

    拦截用户的系统调用，帮助进行调度和垃圾回收


## GMP三个基本结构体
1. G
    
    goroutine

2. M

    work process,内核线程。
    M需要获得P才能运行G.
3. P

    Runable(可运行)的Goroutine队列。数量等于系统CPU逻辑核心数，如果超频CPU，逻辑核心数和物理核心数不一致

## 为什么Go早期没有P？
因为当时没有P，M需要从全局Goroutine队列获取可运行G，防止并发数据竞争，需要加全局锁。并发量大时，锁成为了瓶颈。P维护本地Goroutine队列，解决全局锁问题。

## Gosheduler 的核心思想
1. reuse threads
2. 限制同时（不包含阻塞）运行的线程数量为N，N等于系统CPU核心数
3. 线程私有的可运行队里(runqueues)，可以从其他线程stealing goroutine(偷取其他P的Goroutine)来运行，线程阻塞后可以将 runqueues 传给其他线程。


## 什么是全局可运行队列、本地可运行队列？
- 全局可运行队列

    放在全局可运行队列是因为里面的Goroutine暂时无法分配P
- 本地可运行队列 

    Runable(可运行)的Goroutine队列。数量等于系统CPU逻辑核心数，如果超频CPU，逻辑核心数和物理核心数不一致

# M寻找Goroutine
1. 先从P本地队列找
2. Goroutine 全局队列

    如果全局队列有goroutine，根据全局可运行goroutine的队列长度和 P 的总数计算每个P可分到的平均数量。
3. 从其他P队列偷，取半。
4. 如果找不到，进去休眠状态。

# goroutine 调度时机
1. 使用go 关键字
    
    新任务 goroutine

2. GC
3. 系统调用

    当 goroutine 进行系统调用时，会阻塞 M，所以它会被调度走，同时一个新的 goroutine 会被调度上来
4. 内存同步访问

    atomic、channel、mutex 等操作使goroutine阻塞，会被调度走
## Goroutine的状态
- Waiting 等待调度
- Runnable 就绪状态(等待M执行)
- Executing 正在执行(在 M 上执行指令)

![image](/img/goroutine-workflow.png)

## Goroutine 和线程的区别
1. 内存占用
    - Goroutine 栈内存消耗2kb、栈空间不够用可动态扩容
    - thread 消耗1MB栈内存
2. 创建和销毁
    - Goroutine由Goruntime负责管理，用户级别
    - Thread 创建和销毁需要进行系统调用、内核级别（可以用线程池解决）
3. 切换
    - Goroutine 只需保存三个寄存器 Program counter、strack pointer、BP，切换时间为200ns
    - Thread 保存系统级别寄存器、切换时间为1000-1500ns


