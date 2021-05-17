# TCP 解析

## 1. TCP 头部字段
<img src="http://qt8wkgtri.hd-bkt.clouddn.com/blog/tcp-header.png" alt="tcp-header" style="zoom:50%;" />



### 头部字段作用

- 源端口号
- 目标端口号
  - 找到应用进程
- 序列号
  - 解决网络包乱序的问题
- 确认应答号
  - 用来解决丢包的问题
- 首部长度
  - 获取tcp包头的大小，并计算TCP包数据的长度
- 控制位
  - SYN
    - 为1时，表示建立链接；并在序列号字段填充初始值
  - ACK
    - 为1时，表示确认应答
  - FIN
    - 为1时，表示之后不会再有数据发送，并希望断开连接
  - RST
    - 为1时，表示TCP连接异常强制断开连接
- 窗口大小
  - 表示tcp 连接的滑动窗口可接受数据窗口大小
- 校验和
- 紧急指针

## 2. 什么是TCP 四元组？

包括以下信息

- 源IP地址（在IP包中）
- 目标IP地址（在IP包中）
- 源端口号（在TCP包头中）
- 目标端口号（在TCP包头中）

通过TCP四元组可以确定一个TCP连接。

## 3. 什么是TCP的可靠性传输？

- 序列号
- 确认应答机制
- 重发机制
- TCP连接管理
- 窗口控制

## 4. TCP的序列号怎么生成的？
- 第一个包是随机数(建立连接的时候)
- 如果当前包payload数据为100字节，拿下一个包seq为101

## 5. TCP 三次握手和四次挥手
### 三次握手
<img src="/Users/fotoable/GolandProjects/github.com/chen-shiwei.github.io/docs/post/CS/计算机网络/TCP解析.assets/三次握手.jpg" alt="三次握手" style="zoom:62%;" />
### 四次挥手

<img src="/Users/fotoable/GolandProjects/github.com/chen-shiwei.github.io/docs/post/CS/计算机网络/TCP解析.assets/四次挥手.png" alt="四次挥手" style="zoom:62%;" />

