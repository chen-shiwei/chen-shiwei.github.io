---
title: "context"
date: "2020-12-22 18:04:12"
tag: ["Golang"]
categories: ["Golang","context"]
draft: true
---

## 一.context 作用 

​	在goroutine形成的树形结构中进行信号同步以减少计算资源的浪费

- 在goroutine 同步特定数据
- 取消信号
- 处理请求的截止日期

## 二.context.Background()、conetxt.TODO() 区别

```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}
```

- 都是由emptyCtx 实现的
- 使用语义不同
  - background 
    - 上下文默认值，衍生所有上下文
  - todo
    - 不缺使用那种时，就选todo

## 三. 用法

- context.WithCancel() 传递取消信号	
  - context.WithDeadline()
  - context.WithTimeout()
- context.WithValue() 上下文传值