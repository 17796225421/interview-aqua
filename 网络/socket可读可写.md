**socket可读的条件：**

一、下列四个条件中的任何一个满足时,socket准备好读: 
\1. socket的接收缓冲区中的数据字节大于等于该socket的接收缓冲区低水位标记的当前大小。对这样的socket的读操作将不阻塞并返回一个大于0的值(也就是返回准备好读入的数据)。我们可以用SO_RCVLOWATsocket选项来设置该socket的低水位标记。对于TCP和UDP .socket而言，其缺省值为1.
\2. 该连接的读这一半关闭(也就是接收了FIN的TCP连接)。对这样的socket的读操作将不阻塞并返回0
3.socket是一个用于监听的socket,并且已经完成的连接数为非0.这样的soocket处于可读状态,是因为socket收到了对方的connect请求,执行了三次握手的第一步:对方发送SYN请求过来,使监听socket处于可读状态;正常情况下,这样的socket上的accept操作不会阻塞;
4.有一个socket有异常错误条件待处理.对于这样的socket的读操作将不会阻塞,并且返回一个错误(-1),errno则设置成明确的错误条件.这些待处理的错误也可通过指定socket选项SO_ERROR调用getsockopt来取得并清除;

 

**socket可写的条件：**

二、下列三个条件中的任何一个满足时,socket准备好写: 

\1. socket的发送缓冲区中的数据字节大于等于该socket的发送缓冲区低水位标记的当前大小。对这样的socket的写操作将不阻塞并返回一个大于0的值(也就是返回准备好写入的数据)。我们可以用SO_SNDLOWAT socket选项来设置该socket的低水位标记。对于TCP和UDPsocket而言，其缺省值为2048

\2. 该连接的写这一半关闭。对这样的socket的写操作将产生SIGPIPE信号，该信号的缺省行为是终止进程。

3.有一个socket异常错误条件待处理.对于这样的socket的写操作将不会阻塞并且返回一个错误(-1),errno则设置成明确的错误条件.这些待处理的错误也可以通过指定socket选项SO_ERROR调用getsockopt函数来取得并清除;