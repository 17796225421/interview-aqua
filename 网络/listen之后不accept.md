# socket程序listen之后，不写accept函数，会是怎样一种情况？

```csharp
int listen(int sockfd, int backlog);
```

backlog,就是默认最大的等待队列长度


不用accept 最大的监听个数就是这个backlog了,因为一直没有accept，所以这个里的队列会一直在，不会减少虽然这个backlog是可以手动设置的，但是[linux](https://so.csdn.net/so/search?from=pc_blog_highlight&q=linux)是最大值128,如果超过这个数目了，就是无效的，最大也就128.

在被动状态的socket有两个队列，一个是正在进行三次握手的socket队列，一个是完成三次握手的socket队列。在握手完成后会从正在握手队列移到握手完成的队列，此时已经建立连接。accept就是从已经完成三次握手的socket队列里面取，不accept客户端能完成的连接就是此队列的大小.

所以，不accpet只是不能继续通讯而已，三次握手是协议栈做的，不是api控制的，否则，你考虑效率要多低，就好像每次这种贴都会争论什么阻塞send什么时候返回，阻塞recv什么时候返回。

listen()维护两个队列,一个是未完成三次握手的,一个是已完成三次握手的,accept()是从已完成三次握手的队列中取出一个而已！

当客户端连接服务端后, 通过 netstat 看到连接状态为 ESTABLISHED, 这说明 TCP 三次握手已经成功, 也就是说 TCP 连接已经在网络上建立了起来. 可得知 TCP 握手并不是 Accept 函数的职责.

阅读操作系统的 Accept 函数文档: http://man7.org/linux/man-pages/man2/accept.2.html, 在第一段落中有如下描述:

> It extracts the first connection request on the queue of pending connections for the listening socket, sockfd, creates a new connected socket, and returns a new file descriptor referring to that socket.
>
> 翻译: 它从 connections 队列中取出第一个 connection, 并返回引用该 connection 的一个新的文件描述符.

验证了我的想法, 无论是否调用 Accept, connection 都已经建立起来了, Accept 只是将该 connection 包装成一个文件描述符, 供程序 Read, Write 和 Close. 