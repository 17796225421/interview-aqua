你连 UDP 还是 TCP 都没说。

- 对于 UDP，多线程读写同一个 socket 不用加锁，不过更好的做法是每个线程有自己的 socket，避免 contention，可以用 SO_REUSEPORT 来实现这一点。
- 对于 TCP，通常多线程读写同一个 socket 是错误的设计，因为有 short write 的可能。假如你加锁，而又发生 short write，你是不是要一直等到整条消息发送完才解锁（无论阻塞IO还是非阻塞IO）？如果这样，你的临界区长度由对方什么时候接收数据来决定，一个慢的 peer 就把你的程序搞死了。

总结：对于 UDP，加锁是多余的；对于 TCP，加锁是错误的。

short write： write(2)/send(2) 返回的数字比你要发送的字节数小。