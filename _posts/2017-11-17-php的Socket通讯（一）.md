---
layout:     post
title:      php的Socket通讯（一）
subtitle:   php 进阶
date:       2017-11-17
author:     feimo
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - PHP
    - Socket
---
TCP/IP、UDP、Socket编程这些词你不会很陌生吧？随着网络技术的发展，这些词充斥着我们的耳朵。那么我想问：
  1. 什么是TCP/IP、UDP？
  2. Socket在哪里呢？
  3. Socket是什么呢？
  4. 你会使用它们吗？
# 什么是TCP/IP、UDP？
&emsp;TCP/IP（Transmission Control Protocol/Internet Protocol）即传输控制协议/网间协议，是一个工业标准的协议集，它是为广域网（WANs）设计的。UDP（User Data Protocol，用户数据报协议）是与TCP相对应的协议。它是属于TCP/IP协议族中的一种。
&emsp;这里有一张图，表明了这些协议的关系:
![](https://i.imgur.com/XkhhZnr.jpg)

TCP/IP协议族包括运输层、网络层、链路层。现在你知道TCP/IP与UDP的关系了吧

# Socket在哪里呢？
&emsp;在图1中，我们没有看到Socket的影子，那么它到底在哪里呢？还是用图来说话，一目了然:
![](https://i.imgur.com/OAU1oCN.jpg)
# Socket是什么呢？
　Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。
你会使用它们吗？
　前人已经给我们做了好多的事了，网络间的通信也就简单了许多，但毕竟还是有挺多工作要做的。以前听到Socket编程，觉得它是比较高深的编程知识，但是只要弄清Socket编程的工作原理，神秘的面纱也就揭开了。
　一个生活中的场景。你要打电话给一个朋友，先拨号，朋友听到电话铃声后提起电话，这时你和你的朋友就建立起了连接，就可以讲话了。等交流结束，挂断电话结束此次交谈。 生活中的场景就解释了这工作原理，也许TCP/IP协议族就是诞生于生活中，这也不一定。
![](https://i.imgur.com/l3DctvC.jpg)

先从服务器端说起。服务器端先初始化Socket，然后与端口绑定(bind)，对端口进行监听(listen)，调用accept阻塞，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器(connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。
# socket相关函数：
```php
socket_accept() 接受一个Socket连接
socket_bind() 把socket绑定在一个IP地址和端口上
socket_clear_error() 清除socket的错误或者最后的错误代码
socket_close() 关闭一个socket资源
socket_connect() 开始一个socket连接
socket_create_listen() 在指定端口打开一个socket监听
socket_create_pair() 产生一对没有区别的socket到一个数组里
socket_create() 产生一个socket,相当于产生一个socket的数据结构
socket_get_option() 获取socket选项
socket_getpeername() 获取远程类似主机的ip地址
socket_getsockname() 获取本地socket的ip地址
socket_iovec_add() 添加一个新的向量到一个分散/聚合的数组
socket_iovec_alloc() 这个函数创建一个能够发送接收读写的iovec数据结构
socket_iovec_delete() 删除一个已经分配的iovec
socket_iovec_fetch() 返回指定的iovec资源的数据
socket_iovec_free() 释放一个iovec资源
socket_iovec_set() 设置iovec的数据新值
socket_last_error() 获取当前socket的最后错误代码
socket_listen() 监听由指定socket的所有连接
socket_read() 读取指定长度的数据
socket_readv() 读取从分散/聚合数组过来的数据
socket_recv() 从socket里结束数据到缓存
socket_recvfrom() 接受数据从指定的socket，如果没有指定则默认当前socket
socket_recvmsg() 从iovec里接受消息
socket_select() 多路选择
socket_send() 这个函数发送数据到已连接的socket
socket_sendmsg() 发送消息到socket
socket_sendto() 发送消息到指定地址的socket
socket_set_block() 在socket里设置为块模式
socket_set_nonblock() socket里设置为非块模式
socket_set_option() 设置socket选项
socket_shutdown() 这个函数允许你关闭读、写、或者指定的socket
socket_strerror() 返回指定错误号的详细错误
socket_write() 写数据到socket缓存
socket_writev() 写数据到分散/聚合数组
```
> 原文链接 [(http://www.cnblogs.com/thinksasa/archive/2013/02/26/2934206.html)](http://www.cnblogs.com/thinksasa/archive/2013/02/26/2934206.html)
        