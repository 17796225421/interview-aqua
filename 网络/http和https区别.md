#### HTTP与HTTPS的区别是什么

- HTTP端口号80端口， HTTPS端口号是443端口
- HTTP是明文传输，HTTPS是密文传输。
- HTTP 连接建立相对简单， TCP 三次握手之后便可进行 HTTP 的报文传输。而 HTTPS 在 TCP三次握手之后，还需进行 SSL/TLS 的握手过程，才可进入加密报文传输
- HTTPS 协议需要向 CA（证书权威机构）申请数字证书，来保证服务器的身份是可信的。