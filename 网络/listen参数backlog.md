- *listen* 函数的 *backlog* 的意义

  *Linux*内核中会维护两个队列，*backlog* 大小就是指定的已完成连接队列的大小

  - 未完成连接队列（***SYN*** 队列）：接收到一个 SYN 建立连接请求，处于 ***SYN_RCVD*** 状态
  - 已完成连接队列（***Accpet*** 队列）：已完成 TCP 三次握手过程，处于 ***ESTABLISHED*** 状态