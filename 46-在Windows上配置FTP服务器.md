<!--markdown-->
FTP是一个多端口协议，分别是控制端口和数据端口。
控制端口是TCP 21，数据端口是随机的。

# 设置被动连接的数据端口

在服务器主页设置FTP的被动数据端口，如下图，这些端口还需要在防火墙中放行。

![FTP设置](https://raw.githubusercontent.com/zyywo/zyywo.pic/master/FTP%E8%AE%BE%E7%BD%AE.png)

防火墙的外部IP是公网IP，被动模式下，服务器会把这个IP和数据通道的端口号一起发送给客户端，客户端访问这个IP和端口来传输数据。

更改端口号和外部IP后，需要重启服务器才能生效。


	