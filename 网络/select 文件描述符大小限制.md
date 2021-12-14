# select的用法

select模型的典型使用方法如下：

```c
    while (1) {       
        fd_set rfds;
        fd_set wfds;
        int32_t maxfd = 0, res = 0;
        struct timeval timeout;
        
        timeout.tv_sec = 0;
        timeout.tv_usec = 500;
        
        FD_ZERO(&rfds);
        FD_ZERO(&wfds);
        
        FD_SET(socket1, &rfds);
        FD_SET(socket2, &rfds);
        
        maxfd = socket1 > socket2 ? socket1 : socket2;
        
        res = select(maxfd + 1, &rfds, NULL, NULL, &timeout);
        
        if (res < 0 && errno != EINTR && errno != 0) {
            // log it
            return;
        }
        
        if (FD_ISSET(socket1, &rfds)) {
            // do something 
        }
        if (FD_ISSET(socket2, &rfds)) {
            // do something 
        }
     }
```

select的第一个参数为输入参数，其它4个参数既是输入也是输出。3个事件集合：读事件集合、写事件集合、异常事件集合。输出为触发了该事件的集合。最后一个参数为还剩余多少时间，如果timeout了，则其为0。

# select的几个缺点

## 1.fd_set容量

fd_set定义如下：

```c
typedef __kernel_fd_set		fd_set;

#define __FD_SETSIZE	1024

typedef struct {
	unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))];
} __kernel_fd_set;
```

可见fd_set是一个 __FD_SETSIZE bits大小的数组，每个位对应一个句柄，系统默认情况下select只支持1024个socket。

当然可以通过修改内核来改变这一默认大小。

进程的文件描述符上限是可以手动修改的

```shell
# 查看进程的文件描述符上限
ulimit -n

# 修改进程的文件描述符上限为2048，临时修改，只对当前shell有效
ulimit -SHn 2048

# 永久修改：编辑/etc/security/limits.conf
vi /etc/security/limits.conf
* hard nofile 65536
* soft nofile 65536
```

**但是select的数组大小改不了 (╯‵□′)╯︵┻━┻**，要改只能重新编译内核

但实际上呢？内核并不关心这一数组的大小，内核在分配空间时使用的是select的第一个参数(最大的fd)来计算的，具体代码如下：

```c
	/* max_fds can increase, so grab it once to avoid race */
	rcu_read_lock();
	fdt = files_fdtable(current->files);
	max_fds = fdt->max_fds;
	rcu_read_unlock();
	if (n > max_fds)
		n = max_fds;

	/*
	 * We need 6 bitmaps (in/out/ex for both incoming and outgoing),
	 * since we used fdset we need to allocate memory in units of
	 * long-words. 
	 */
	size = FDS_BYTES(n);
	bits = stack_fds;
	if (size > sizeof(stack_fds) / 6) {
		/* Not enough space in on-stack array; must use kmalloc */
		ret = -ENOMEM;
		bits = kmalloc(6 * size, GFP_KERNEL);
		if (!bits)
			goto out_nofds;
	}
	fds.in      = bits;
	fds.out     = bits +   size;
	fds.ex      = bits + 2*size;
	fds.res_in  = bits + 3*size;
	fds.res_out = bits + 4*size;
	fds.res_ex  = bits + 5*size;
  if ((ret = get_fd_set(n, inp, fds.in)) ||
	    (ret = get_fd_set(n, outp, fds.out)) ||
	    (ret = get_fd_set(n, exp, fds.ex)))
		goto out;
	zero_fd_set(n, fds.res_in);
	zero_fd_set(n, fds.res_out);
	zero_fd_set(n, fds.res_ex);

	ret = do_select(n, &fds, end_time);
```

可以看到，分配的内核空间bits只和传入的第一参数有关，取传入的参数和该进程支持的最大句柄的最小值。

## 2.句柄过大的问题

如果一个应用程序通过setrlimit把进程可打开的最大fd(RLIMIT_NOFILE)改成2048，而__FD_SETSIZE是默认的1024。此时打开很多文件，最后才创建了一个socket，fd是1500，这个时候会有什么问题呢？

很明显，1500这个fd是设置不进去fd_set里去的，会越界。(可以看下FD_SET的实现，这种越界并不会导致程序崩溃，不设该位而已)

select在执行过程中，会先把用户态的fd_set拷贝到内核态，也就是上面代码中的get_fd_set那三个操作。

此时传入select的第一个参数应该是1501，内核会依此分配空间，并从用户态拷贝1501个bits。但超过__FD_SETSIZE 的部分内存是未初始化的，这样内核就会拷贝一个我们不期望的fd_set，未初始化的内存可能是0，可能是1，这就意味着我们监控了我们不希望监控的fd，而这些句柄恰好又都是存在的(因为我们放大了RLIMT_NOFILE并打开了很多文件)。这些1500以后的句柄一有事件，select就会返回。这样很容易造成死循环：select不断触发但后面判断FD_ISSET又不成立。



## 3.性能问题

下面是select最核心的代码实现：

```c
for (;;) {
    ...
    for (i = 0; i < n; ++rinp, ++routp, ++rexp) {
        ...
        for (j = 0; j < BITS_PER_LONG; ++j, ++i, bit <<= 1) {
            ...
            if (!(bit & all_bits))
                continue;
            file = fget_light(i, &fput_needed);
            if (file) {
            		f_op = file->f_op;
								mask = DEFAULT_POLLMASK;
								if (f_op && f_op->poll) {
										wait_key_set(wait, in, out, bit);
										mask = (*f_op->poll)(file, wait);
								}
								fput_light(file, fput_needed);
								if ((mask & POLLIN_SET) && (in & bit)) {
										res_in |= bit;
										retval++;
										wait->_qproc = NULL;
								}
								if ((mask & POLLOUT_SET) && (out & bit)) {
										res_out |= bit;
									  retval++;
									  wait->_qproc = NULL;
								}
								if ((mask & POLLEX_SET) && (ex & bit)) {
						        res_ex |= bit;
						        retval++;
						        wait->_qproc = NULL;
					      }
					 }
					 cond_resched();
				}
			  wait->_qproc = NULL;
				if (retval || timed_out || signal_pending(current))
					 break;
				if (table.error) {
			     retval = table.error;
			     break;
		    }

				/*
		 				* If this is the first loop and we have a timeout
						* given, then we convert to ktime_t and set the to
		 				* pointer to the expiry value.
		 		*/
				if (end_time && !to) {
					 expire = timespec_to_ktime(*end_time);
			     to = &expire;
				}

				if (!poll_schedule_timeout(&table, TASK_INTERRUPTIBLE,
					   		to, slack))
							timed_out = 1;
}
```

可以看到，select是遍历fd_set中为1的fd，不管是读fd_set，还是写fd_set，还是异常fd_set。如果为1，则调相应设备的pool_wait查看是否有事件，并且将该进程加入相应设备的等待队列。遍历完一次fd_set之后，就进入阻塞等待，直到timeout，或有设备通知事件到来唤醒该进程。

这样，fd_set越大，其性能就越低，而且fd_set很大的时候，fd_set在用户态和内核态之间的拷贝也是很耗时的。所以很多文章里都提到，fd_set不应该超过64，超过64应该考虑使用epoll。