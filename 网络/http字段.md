### HTTP 字段

- ***Host***: 客户端发送请求时，用来指定服务器的域名。

- ***Content-Length***：服务器在返回数据时，会有 Content-Length 字段，表明本次回应的数据长度。

- ***Connection***：Connection 字段最常用于客户端要求服务器使用 TCP 持久连接，以便其他请求复用。

  > HTTP/1.1 版本的默认连接都是持久连接，但为了兼容老版本的 HTTP，需要指定 Connection 首部字段的值为 Keep-Alive。
  >
  > *Connection: keep-alive*

- ***Content-Type***：用于服务器回应时，告诉客户端本次数据是什么格式。

  > Content-Type: text/html; charset=utf-8 ：表明，发送的是网页，而且编码是UTF-8。
  >
  > 客户端请求的时候，可以使用 **Accept** 字段声明自己可以接受哪些数据格式。比如：
  > Accept: */* ：客户端声明自己可以接受任何格式的数据。

- ***Content-Encoding***：说明数据的压缩方法。表示服务器返回的数据使用了什么压缩格式

  > 客户端在请求时，用 **Accept-Encoding** 字段说明自己可以接受哪些压缩方法。