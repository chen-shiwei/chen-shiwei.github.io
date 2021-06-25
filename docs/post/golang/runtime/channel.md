## channel

goroutine 之间的通信方式

# 设计原理

## 设计模式

- 不要通过共享内存来通信，而是通过通信共享内存。

- CSP(Communicating sequential processes)并发模型，通信顺序进程
- 先进新出
  - 先从channel 读取数据的goroutine会先接收到数据
  - 先从channel 发送数据的goroutine会得到先发送数据的权利

## 无锁管道

Channel 运行时的内部，是一个用于同步和通信的有锁队列，使用互斥锁解决程序中可能存在的线程竞争问题

```go
// https://github.com/golang/go/blob/41d8e61a6b9d8f9db912626eb2bbc535e929fefc/src/runtime/chan.go#L32
type hchan struct {
	qcount   uint           // total data in the queue channel中的元素数量
	dataqsiz uint           // size of the circular queue  channel循环队列的长度
	buf      unsafe.Pointer // points to an array of dataqsiz elements 缓冲区数据指针
	elemsize uint16 // 元素的大小
	closed   uint32 
	elemtype *_type // element type 元素类型
	sendx    uint   // send index channel 发送操作的位置
	recvx    uint   // receive index channel 接收的位置
	recvq    waitq  // list of recv waiters 由于缓冲区空间不足阻塞的goroutine列表 双链表
	sendq    waitq  // list of send waiters 由于缓冲区空间不足阻塞的goroutine列表 双链表

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}


type waitq struct {
	first *sudog
	last  *sudog
}
```

## 创建 channel

```go
func makechan(t *chantype, size int) *hchan {
	elem := t.elem
	mem, _ := math.MulUintptr(elem.size, uintptr(size))

	var c *hchan
	switch {
	case mem == 0:
		c = (*hchan)(mallocgc(hchanSize, nil, true))
		c.buf = c.raceaddr()
	case elem.kind&kindNoPointers != 0:
		c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
		c.buf = add(unsafe.Pointer(c), hchanSize)
	default:
		c = new(hchan)
		c.buf = mallocgc(mem, elem, true)
	}
	c.elemsize = uint16(elem.size)
	c.elemtype = elem
	c.dataqsiz = uint(size)
	return c
}
```

## channel 发送

![channel-direct-send](https://img.draveness.me/2020-01-29-15802354027250-channel-direct-send.png)

发送数据时调用 [`runtime.send`](https://draveness.me/golang/tree/runtime.send)

- 调用 [`runtime.sendDirect`](https://draveness.me/golang/tree/runtime.sendDirect) 将发送的数据直接拷贝到 `x = <-c` 表达式中变量 `x` 所在的内存地址上；
- 调用 [`runtime.goready`](https://draveness.me/golang/tree/runtime.goready) 将等待接收数据的 Goroutine 标记成可运行状态 `Grunnable` 并把该 Goroutine 放到发送方所在的处理器的 `runnext` 上等待执行，该处理器在下一次调度时会立刻唤醒数据的接收方(放置到runnext)；

## channel接收

```go
i <- ch
i, ok <- ch
```

## channel 接收数据时，会触发goroutine调度时机

- channel 为空时
- 当缓冲区不存在数据也不存在数据的发送者时