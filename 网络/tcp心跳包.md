#### 保活机制与心跳包

心跳检测一般有两个作用：

- 保活：不想让长时间没有数据传输的连接被防火墙关闭，那么就需要 **保活**
- 检测死链：因网络故障导致连接没有数据传输，无论是客户端或者服务器都无法感知与对方的连接是否正常，这类连接我们一般称之为“死链”

##### SO_KEEPLIVE

设置 `KeepLive`选项参数 `SO_KEEPALIVE` 来保活。这个选项默认发送心跳检测数据包的时间间隔是 7200 秒（2 小时）

当然，我们可以通过继续设置 `keepalive` 相关的三个选项来改变这个时间间隔，它们分别是 `TCP_KEEPIDLE` 、 `TCP_KEEPINTVL` 和 `TCP_KEEPCNT`，示例代码如下：

```cpp
  //发送 keepalive 报文的时间间隔
  int val = 7200;
  setsockopt(fd, IPPROTO_TCP, TCP_KEEPIDLE, &val, sizeof(val));

  //两次重试报文的时间间隔
  int interval = 75;
  setsockopt(fd, IPPROTO_TCP, TCP_KEEPINTVL, &interval, sizeof(interval));
  
  // 探测几次
  int cnt = 9;
  setsockopt(fd, IPPROTO_TCP, TCP_KEEPCNT, &cnt, sizeof(cnt));
```

`TCP_KEEPIDLE` 选项设置了发送 `keepalive` 报文的时间间隔，发送时如果对端回复 `ACK` 。则本端 TCP 协议栈认为该连接依然存活，继续等 7200 秒后再发送 `keepalive` 报文；如果对端回复 `RST`，说明对端进程已经重启，本端的应用程序应该关闭该连接。

如果对端没有任何回复，则本端做重试，如果重试 9 次（ `TCP_KEEPCNT` 值）（前后重试间隔为 75 秒（ `TCP_KEEPINTVL` 值））仍然不可达，则向应用程序返回 `ETIMEOUT` （无任何应答）或 `EHOST` 错误信息

##### 应用层的心跳包机制设计

由于 `keepalive` 选项需要为每个连接中的 `socket` 开启，由于这不一定是必须的，可能会产生大量无意义的带宽浪费，且 `keepalive` 选项不能与应用层很好地交互，因此一般实际的服务开发中，还是建议读者在应用层设计自己的心跳包机制。