假如我调用了一个 select 函数，并且关注了几个描述字， select 函数就会一直阻塞直到我关注的事件发生. 假如当有套接口可读时， select 函数就返回了，告诉我们套接口已经可读，然后我们去读这个套接口，可以用阻塞的read或者非阻塞的 read，阻塞 read 是无数据可读就阻塞进程，非阻塞 read是无数据可读就返回一个 EWOULDBLOCK 错误。那么问题来了：既然 select 都返回可读了，那就表示一定能读了，阻塞函数read也就能读取了也就不会阻塞了，非阻塞read的话，也有数据读了，也不会返回错误了，那么这俩不都一样了？一样直接读取数据知道读完，为什么还得用非阻塞函数？还有 Reactor 模式也是用的 IO 多路复用与非阻塞 IO，这是什么道理呢？

man 2 select 「BUGS」节：

Under Linux, select() may report a socket file descriptor as "ready for reading", while nevertheless a subsequent read blocks. This could for example happen when

data has arrived but upon examination has wrong checksum and is discarded. There may be other circumstances in which a file descriptor is spuriously reported as

ready. Thus it may be safer to use O_NONBLOCK on sockets that should not block.

就算数据不会被别人读走，也可能被内核丢弃。还有文档没有明说的其它情况。

假如 socket 的读缓冲区中已有足够多的数据，需要调用三次 read 才能读取完。或 ACCEPT 队列已经有三个「握手已完成的连接」。

非阻塞 I/O 的处理方式：循环的 read 或 accept，直到读完所有的数据（抛出 EWOULDBLOCK 异常）。

阻塞 I/O 的处理方式：每次只能调用一次 read 或 accept，***因为多路复用只会告诉你 fd 对应的 socket 可读了，但不会告诉你有多少的数据可读\***，所以在 handle_read/handle_accept 中只能 read/accept 一次，你无法知道下一次 read/accept 会不会发生阻塞。所以只能等 ioloop 的第二次循环，ioloop 告诉你 fd 可用后再继续调用 handle_read/handle_accept 处理，然后再循环第三次。

所以你会发现，后者的处理方式要复杂很多，稍不注意就会阻塞整个进程。