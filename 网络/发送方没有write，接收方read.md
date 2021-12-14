## read/write 回溯

首先，write成功返回，**只是buf中的数据被复制到了kernel中的TCP发送缓冲区。**至于数据什么时候被发往网络，什么时候被对方主机接收，什么时候被对方进程读取，系统调用层面不会给予任何保证和通知。其实这里面涉及传输层以下的协议了，之前我一直以为只要write进去就是发过去了，其实这个是错误的认知，发不发是系统决定的。涉及到发送有个算法决定，Nagle算法是被TCP默认开启的，这里涉及一个字段选项设置，TCP_NODELAY，启用这个字段就把Nalge算法禁止掉了。如果读者有兴趣可以自行百度这两个关键字。write在什么情况下会阻塞？当kernel的该socket的发送缓冲区已满时。对于每个socket，拥有自己的send buffer和[receive buffer](https://www.zhihu.com/search?q=receive+buffer&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A71799852})。从Linux 2.6开始，两个缓冲区大小都由系统来自动调节（autotuning），但一般在default和max之间浮动。

```text
# 获取socket的发送/接受缓冲区的大小：（后面的值是在我在Linux 2.6.38 x86_64上测试的结果）
sysctl net.core.wmem_default       #126976
sysctl net.core.wmem_max　　　　    #131071
sysctl net.core.wmem_default       #126976
sysctl net.core.wmem_max           #131071
```

已经发送到网络的数据依然需要暂存在send buffer中，只有收到对方的ack后，kernel才从buffer中清除这一部分数据，为后续发送数据腾出空间。接收端将收到的数据暂存在receive buffer中，自动进行确认。但如果socket所在的进程不及时将数据从receive buffer中取出，最终导致receive buffer填满，由于TCP的滑动窗口和拥塞控制，[接收端](https://www.zhihu.com/search?q=接收端&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A71799852})会阻止发送端向其发送数据。这些控制皆发生在TCP/IP栈中，对应用程序是透明的，应用程序继续发送数据，最终导致send buffer填满，write调用阻塞。一般来说，由于接收端进程从socket读数据的速度跟不上发送端进程向socket写数据的速度，最终导致发送端write调用阻塞。read调用的行为相对容易理解，从socket的receive buffer中拷贝数据到应用程序的buffer中。read调用阻塞，通常是发送端的数据没有到达。