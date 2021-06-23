![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0003-squashed.jpg)
# 1. http2 协议
http2 是grpc 的网络传输层
设备之前网络一个请求和响应是一个rtt，tcp 连接需要2个rtt，http跳转https需要三次握手开销

## 1.1grpc为什么选择http2
兼容，主流proxy、loadbalancer兼容http2，也就兼容grpc

## 1.2 http 发展
- http 1.0
	- 缺点短连接
- http 1.1
	- 解决短连接问题，增加丰富的header语义
	- 缺点
		- 队头阻塞，一个http连接只能处理一个请求，没有返回时，其他请求无法占用
		- 连接无法复用
		- http 1.1 pipline 多数浏览器不支持
- http 2
	- header 压缩，减少流量使用
	
	- 流控
	
	- 多路复用
	
	- 协议层复用，把一个个请求封装城城steam流，流并发交错请求
  
  - 服务端推送
  
  - 优先级
  
  - 帧的格式
    
    - Length(24) payload的长度
    - Type(8)
      
      - HEADERS
        
	    - DATA
	      
	    - PRIORITY
	      
	    - RST_STREAM
	      
	    - SETTINGS
	      
	    - PUSH_PROMISE
	      
	    - PING
	      
	    - GOAWAY
	      
	    - WINDOW_UPDATE
	      
	    - CONTINUATION
	  - Flags(8)
	  - Stream Identifier(31)
	  - R 保留字段
	  - Frame Payload
	      ![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0022-squashed.jpg)
# 2. protobuf 编码
## 2.1 为什么选择protobuf
- json
  - waste(浪费) 空格
  - lost type 丢失类似 float => int
  - 不安全
  - 性能低 明文
  - {} "" [] , 作为分隔符


- protobuf

  - 二进制，节省空间传更多数据
- by google
  - TLV![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0036-squashed.jpg)
- 序列化性能高

![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0032-squashed.jpg)

# 3. grpc
## 3.2 请求类型

- unary 请求响应
- client stream
- server stream
- Bidi stream 双向流



## 3.3 header frame

- 元数据在header 里面

- 服务 method

- router

  ![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0052-squashed.jpg)
## 3.4 Stream frame 存储 protobuf 编码数据

![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0053-squashed.jpg)

- header frame

- Content-type: application/grpc

- grpc-status

  ![img](http://xiaorui.cc/wp-content/uploads/2020/08/grpc-protobuf-http2_compressed_page-0055-squashed.jpg)

- grpc-message
## 3.5 grpc 相关工具
- bloomrpc gui
- ghz benchmark