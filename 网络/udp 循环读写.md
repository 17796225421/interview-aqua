​	疑问：有时候会看到某些代码，`sendto()`时用了`while`循环, 而`recvfrom()`时没使用`while`循环？

答：他们都可以使用循环语句，可参考[TCP数据粘包的处理](https://subingwen.cn/linux/tcp-data-package/)。

什么时候需要使用循环，什么时候不使用循环，可以看下面的分析：

##### 以下其实是我根据自己项目使用的`udp`协议中的`recvfrom()`和`sendto()`进行测试没问题后分析的。但是对于`TCP`粘包的问题，却并非如此，并非`recv`每次只取一次完整发送的数据（`UDP`的`recvfrom()`为什么可以取这么准？），我目前还没测试。

##### 1. `recvfrom()`要使用与不使用循环的情况:

我们通常指定的接收端一次接收长度都会 `>=` 发送端一次发送的数据长度。通常情况下，我们`发送端`一次发送的数据长度都不会是固定的，所以就需要`接收端`设置一个合适的固定的接收长度，这个固定长度需要`大于等于`发送端一次发送的最大数据长度。

当`recvfrom()`函数指定`buf`的长度后，并且一次`recvfrom()`函数读取到的数据小于指定长度`max_length`（这个是可以保证的），那么：

- 如果能确定每次`recvfrom()`实际读取到的数据`是`发送端一次发送的完整数据，那就不用循环`recvfrom()`。
- 如果每次`recvfrom()`实际读取到的数据`不是`发送端一次发送的完整数据，就需要循环`recvfrom()`。

------

##### 2.`sendto()`要使用与不使用循环的情况:

`sendto()`一般情况下需要使用循环，因为假如一个数据包太大，如长度为`10MB`，一次`sendto()`发送到`输出缓冲区`可能发不完整，此时就需要对`sendto()`使用循环发送，直到把`10MB`的数据都拷贝到输出缓冲区。
`sendto()`函数中参数指定的数据长度，就是本次发送(就是写入输出缓冲区)的数据长度，都会提前计算好之后再填入，每次发送的数据长短可能不一样，所以他就不是固定长度的。
而`recvfrom()`函数中指定的长度是固定的。

------

##### 3.`recvfrom()`和`sendto()`例子：

`recvfrom()`和`sendto()`的第三个参数`len`都是指定第二个参数`buf`的长度。

- 1.`recvfrom()`从输入缓冲区中拷贝数据到应用程序缓冲区`buf`，在此需要指定`buf`的长度。他的长度一般在定义缓冲`buf`时就定下来了，如

```c
constexpr std::size_t kBufferSize = 1024;
...
uint8_t buf[kBufferSize] = {
    0};  //定义buf时，长度也定下来了
std::memset(buf, 0, kBufferSize);
...
length = data_stream_->read(buf, kBufferSize, 0);  
上面的read()函数会调用： 
ret = ::recvfrom(sockfd_, buffer, kBufferSize, 0)
```

- 2.`sendto()`从输出缓冲区中拷贝数据到应用程序缓冲区`buf`，在此需要指定`buf`的长度。他的长度都会提前计算好之后再填入，每次发送的数据长短可能不一样，所以他就不是固定长度的：

```c
constexpr std::size_t kBufferSize = 1024;

 char buf[kBufferSize] = "abcd";
// proto_msg_是一个已经赋值后的protobuf消息的变量
 int proto_msg_length = proto_msg_.ByteSize();   
 
 // 把 proto_msg_ 序列化进buf，从buf第四字节开始，前四个字节为"cidi"
 proto_msg_.SerializeToArray(&buf[4], proto_msg_length);
 int send_size = proto_msg_length + 4;   // 加上"cidi"四个字节

// 虽然 buf有1024字节，但是只将他的前 send_size 字节写入输出缓冲区
  std::size_t   len = data_stream_->write(reinterpret_cast<uint8_t*>(buf),
                           send_size, 0);

上面的write()函数会调用： 
ret = ::sendto(sockfd_, buffer, kBufferSize, 0)
```

------

------

##### `sendto()`

```c
size_t UdpStream::write(const uint8_t* data, size_t length, uint8_t flag) {
    
  size_t total_nsent = 0;
  // if (flag) {
    
  //   peer_sockaddr_.sin_addr.s_addr = htonl(INADDR_BROADCAST);
  // }
  peer_sockaddr_.sin_addr.s_addr = peer_addr_;
  peer_sockaddr_.sin_port = peer_broad_port_;
  SDEBUG << "sendto addr: " << inet_ntoa(peer_sockaddr_.sin_addr)
         << ", port: " << ntohs(peer_sockaddr_.sin_port);
  while (length > 0) {
    
    ssize_t nsent =
        ::sendto(sockfd_, data, length, 0, (struct sockaddr*)&peer_sockaddr_,
                 (socklen_t)sizeof(peer_sockaddr_));
    if (nsent < 0) {
      // error
      if (errno == EINTR) {
    
        continue;
      } else {
    
        // error
        if (errno == EPIPE || errno == ECONNRESET) {
    
          status_ = Stream::Status::DISCONNECTED;
          errno_ = errno;
        } else if (errno != EAGAIN) {
    
          status_ = Stream::Status::ERROR;
          errno_ = errno;
        }
        return total_nsent;
      }
    }

    total_nsent += nsent;
    length -= nsent;
    data += nsent;
  }
  return total_nsent;
}
```

------

##### `recvfrom()`

```c
size_t UdpStream::read(uint8_t* buffer, size_t max_length, uint8_t flag) {
    
  ssize_t ret = 0;
  struct sockaddr_in addrfrom;
  addrfrom.sin_addr.s_addr = htonl(INADDR_ANY);
  if (flag) {
    
    peer_sockaddr_.sin_addr.s_addr = htonl(INADDR_ANY);
  } else {
    
    addrfrom.sin_addr.s_addr = peer_sockaddr_.sin_addr.s_addr;
  }

  while ((ret = ::recvfrom(sockfd_, buffer, max_length, 0,
                           (struct sockaddr*)&peer_sockaddr_,
                           reinterpret_cast<socklen_t*>(&socklenth_))) < 0) {
    
    if (errno == EINTR) {
    
      continue;
    } else {
    
      // error
      if (errno != EAGAIN) {
    
        status_ = Stream::Status::ERROR;
        errno_ = errno;
      }
    }

    return 0;
  }

  // 接收来自本车obu的数据包：0x63,0x69,0x64,0x69分别表示cidi的ASCII码:99,105,100,105
  // 如果不是"cidi",1.如果是单播，就把ip保持为上一次成功单播的ip；
  // 2.如果是广播，就把ip设为0.0.0.0(即htonl(INADDR_ANY))，即本机任意网卡的ip
  if (0x63 != buffer[0] && 0x69 != buffer[1] &&
      0x64 != buffer[2] && 0x69 != buffer[3]) {
    
    peer_sockaddr_.sin_addr.s_addr = addrfrom.sin_addr.s_addr;
  }

  // // 0x60,0x61 分别对应'`'和'a'的ASCII码 96(`),97(a)
  // if (buffer[0] != 0x60 && buffer[1] != 0x61) {
    
  //   peer_sockaddr_.sin_addr.s_addr = addrfrom.sin_addr.s_addr;
  // }

  SDEBUG << "Receive addr: " << inet_ntoa(peer_sockaddr_.sin_addr)
         << ", port: " << ntohs(peer_sockaddr_.sin_port);

  return ret;
}
```