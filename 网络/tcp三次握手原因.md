**三次握手建立连接时，为什么需要第三次？**

- 三次握手才可以阻止历史重复连接的初始化（主要原因）
- 三次握手才可以同步双方的初始序列号
- 三次握手才可以避免资源浪费

三次握手就是防止因为网络阻塞导致的不确定性。

![img](image/tcp三次握手的主要原因.webp)

**[原因1]**：如果因为网络拥堵导致客户端发出的旧 **SYN** 报文比新的 **SYN** 报文先到达，服务端会发出应答 **SYN + ACK**。客户端可以判断 **SYN** 是一个历史连接，在第三次握手中返回 **RST** 给服务端，中断这个连接。如果是两次握手，客户端就没有足够的上下文来判断这个过时的 **SYN** 是否过时，那么就不能中断这次连接。

- 如果是历史连接（序列号过期或超时），则第三次握手发送的报文是**RST** 报文，以此中止历史连接；
- 如果不是历史连接，则第三次发送的报文是 **ACK** 报文，通信双方就会成功建立连接；

**[原因3]**：如果客户端的 **SYN** 阻塞了，客户端会重复发送多次 **SYN** 报文，那么服务器在收到请求后就会建立多个冗余的无效链接，造成不必要的资源浪费。