### 什么是Scoket

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/v2-226e5169b8c0da05cb3605d00269356f_1440w.jpg" alt="img" style="zoom:50%;" />

> **socket**其实就是操作系统提供给程序员操作**网络协议栈**的接口，也就是通过socket接口，来控制协议栈的工作，从而实现网络通信，达到跨主机通信。
>
> Socket是**应用层与TCP/IP协议族通信的中间软件抽象层**，它是一组接口。**在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面**，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。而我们所说的**socket编程指的是利用soket接口来实现自己的业务和协议**。
>
> **Socke接口属于软件抽象层，而socket编程却是标准的应用层开发**





### TCP网络编程

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/v2-7105d213a9207bf0d497455c652df7e2_1440w.jpg" alt="img" style="zoom:50%;" />

基于 TCP 协议的客户端和服务器工作：

- 服务端和客户端初始化 `socket`，得到文件描述符；
- 服务端调用 `bind`，将绑定在 IP 地址和端口;
- 服务端调用 `listen`，进行监听；
- 服务端调用 `accept`，等待客户端连接；
- 客户端调用 `connect`，向服务器端的地址和端口发起连接请求；
- 服务端 `accept` 返回用于传输的 `socket` 的文件描述符；
- 客户端调用 `write` 写入数据；服务端调用 `read` 读取数据；
- 客户端断开连接时，会调用 `close`，那么服务端 `read` 读取数据的时候，就会读取到了 `EOF`，待处理完数据后，服务端调用 `close`，表示连接关闭。