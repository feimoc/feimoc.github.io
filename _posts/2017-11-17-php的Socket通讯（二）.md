---
layout:     post
title:      php的Socket通讯（二）
subtitle:   php 进阶
date:       2017-11-17
author:     feimo
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - PHP
    - Socket
---


### 案例一：socket通信演示
测试之前需要把php.ini中extension=php_sockets.dll打开
#### 服务器端：
```php
<?php
//确保在连接客户端时不会超时
set_time_limit(0);

$ip = '127.0.0.1';
$port = 1935;

/*
 +-------------------------------
 *    @socket通信整个过程
 +-------------------------------
 *    @socket_create
 *    @socket_bind
 *    @socket_listen
 *    @socket_accept
 *    @socket_read
 *    @socket_write
 *    @socket_close
 +--------------------------------
 */

/*----------------    以下操作都是手册上的    -------------------*/
if(($sock = socket_create(AF_INET,SOCK_STREAM,SOL_TCP)) < 0) {
    echo "socket_create() 失败的原因是:".socket_strerror($sock)."\n";
}

if(($ret = socket_bind($sock,$ip,$port)) < 0) {
    echo "socket_bind() 失败的原因是:".socket_strerror($ret)."\n";
}

if(($ret = socket_listen($sock,4)) < 0) {
    echo "socket_listen() 失败的原因是:".socket_strerror($ret)."\n";
}

$count = 0;

do {
    if (($msgsock = socket_accept($sock)) < 0) {
        echo "socket_accept() failed: reason: " . socket_strerror($msgsock) . "\n";
        break;
    } else {
        
        //发到客户端
        $msg ="测试成功！\n";
        socket_write($msgsock, $msg, strlen($msg));
        
        echo "测试成功了啊\n";
        $buf = socket_read($msgsock,8192);
        
        
        $talkback = "收到的信息:$buf\n";
        echo $talkback;
        
        if(++$count >= 5){
            break;
        };
        
    
    }
    //echo $buf;
    socket_close($msgsock);

} while (true);

socket_close($sock);
?>
```
这是socket的服务端代码。然后运行cmd，注意是自己的程序存放路径啊。
![](https://i.imgur.com/JQFoSnE.png)
没有反映，对现在服务端的程序已经开始运行，端口已经开始监听了。运行netstat -ano可以查看端口情况，我的是1935端口
![](https://i.imgur.com/JUw4Gra.png)
看，端口已经处于LISTENING状态了。接下来我们只要运行客户端程序即可连接上。上代码
#### 客户端：
```php
<?php
//确保在连接客户端时不会超时
set_time_limit(0);

$ip = '127.0.0.1';
$port = 1935;

/*
 +-------------------------------
 *    @socket通信整个过程
 +-------------------------------
 *    @socket_create
 *    @socket_bind
 *    @socket_listen
 *    @socket_accept
 *    @socket_read
 *    @socket_write
 *    @socket_close
 +--------------------------------
 */

/*----------------    以下操作都是手册上的    -------------------*/
if(($sock = socket_create(AF_INET,SOCK_STREAM,SOL_TCP)) < 0) {
    echo "socket_create() 失败的原因是:".socket_strerror($sock)."\n";
}

if(($ret = socket_bind($sock,$ip,$port)) < 0) {
    echo "socket_bind() 失败的原因是:".socket_strerror($ret)."\n";
}

if(($ret = socket_listen($sock,4)) < 0) {
    echo "socket_listen() 失败的原因是:".socket_strerror($ret)."\n";
}

$count = 0;

do {
    if (($msgsock = socket_accept($sock)) < 0) {
        echo "socket_accept() failed: reason: " . socket_strerror($msgsock) . "\n";
        break;
    } else {
        
        //发到客户端
        $msg ="测试成功！\n";
        socket_write($msgsock, $msg, strlen($msg));
        
        echo "测试成功了啊\n";
        $buf = socket_read($msgsock,8192);
        
        
        $talkback = "收到的信息:$buf\n";
        echo $talkback;
        
        if(++$count >= 5){
            break;
        };
        
    
    }
    //echo $buf;
    socket_close($msgsock);

} while (true);

socket_close($sock);
?>
```
![](https://i.imgur.com/FSzsnZz.png)
![](https://i.imgur.com/COaXIvV.png)
至此客户端已经连接上服务端了。
### 案例二：代码详解
```php
// 设置一些基本的变量
$host = "192.168.1.99";
$port = 1234;
// 设置超时时间
set_time_limit(0);
// 创建一个Socket
$socket = socket_create(AF_INET, SOCK_STREAM, 0) or die("Could not createsocket\n");
//绑定Socket到端口
$result = socket_bind($socket, $host, $port) or die("Could not bind tosocket\n");
// 开始监听链接
$result = socket_listen($socket, 3) or die("Could not set up socketlistener\n");
// accept incoming connections
// 另一个Socket来处理通信
$spawn = socket_accept($socket) or die("Could not accept incomingconnection\n");
// 获得客户端的输入
$input = socket_read($spawn, 1024) or die("Could not read input\n");
// 清空输入字符串
$input = trim($input);
//处理客户端输入并返回结果
$output = strrev($input) . "\n";
socket_write($spawn, $output, strlen ($output)) or die("Could not write
output\n");
// 关闭sockets
socket_close($spawn);
socket_close($socket);
```
#### 下面是其每一步骤的详细说明:
1. 第一步是建立两个变量来保存Socket运行的服务器的IP地址和端口.你可以设置为你自己的服务器和端口(这个端口可以是1到65535之间的数字),前提是这个端口未被使用.
    ```php
    // 设置两个变量
    $host = "192.168.1.99";
    $port = 1234;
    ```
2. 在服务器端可以使用set_time_out()函数来确保PHP在等待客户端连接时不会超时.
     ```php
       // 超时时间
       set_time_limit(0);
        ```
3. 在前面的基础上,现在该使用socket_creat()函数创建一个Socket了—这个函数返回一个Socket句柄,这个句柄将用在以后所有的函数中.    
    ```php
    // 创建Socket
    $socket = socket_create(AF_INET, SOCK_STREAM, 0) or die("Could not create
    socket\n");
    ```
    第一个参数”AF_INET”用来指定域名;
    第二个参数”SOCK_STREM”告诉函数将创建一个什么类型的Socket(在这个例子中是TCP类型)
    
    因此,如果你想创建一个UDP Socket的话,你可以使用如下的代码:   
     ```php
     // 创建 socket
     $socket = socket_create(AF_INET, SOCK_DGRAM, 0) or die("Could not create
     socket\n");
     ```
4. 一旦创建了一个Socket句柄,下一步就是指定或者绑定它到指定的地址和端口.这可以通过socket_bind()函数来完成.     
    ```php
    // 绑定 socket to 指定地址和端口
    $result = socket_bind($socket, $host, $port) or die("Could not bind to
    socket\n");
    ```
5. 当Socket被创建好并绑定到一个端口后,就可以开始监听外部的连接了.PHP允许你由socket_listen()函数来开始一个监听,同时你可以指定一个数字(在这个例子中就是第二个参数:3)
    ```php
    // 开始监听连接
    $result = socket_listen($socket, 3) or die("Could not set up socket
    listener\n");
    ```
6. 到现在,你的服务器除了等待来自客户端的连接请求外基本上什么也没有做.一旦一个客户端的连接被收到,socket_accept()函数便开始起作用了,它接收连接请求并调用另一个子Socket来处理客户端–服务器间的信息.
    ```php
    //接受请求链接
    // 调用子socket 处理信息
    $spawn = socket_accept($socket) or die("Could not accept incoming
    connection\n");
    ```  
    这个子socket现在就可以被随后的客户端–服务器通信所用了.
7. 当一个连接被建立后,服务器就会等待客户端发送一些输入信息,这写信息可以由socket_read()函数来获得,并把它赋值给PHP的$input变量.    
    ```php
    // 读取客户端输入
    $input = socket_read($spawn, 1024) or die("Could not read input\n");
    ```
   socker_read的第而个参数用以指定读入的字节数,你可以通过它来限制从客户端获取数据的大小.
   
   注意:socket_read函数会一直读取壳户端数据,直到遇见\n,\t或者\0字符.PHP脚本把这写字符看做是输入的结束符.
8. 现在服务器必须处理这些由客户端发来是数据(在这个例子中的处理仅仅包含数据的输入和回传到客户端).这部分可以由socket_write()函数来完成(使得由通信socket发回一个数据流到客户端成为可能)
    ```php
    // 处理客户端输入并返回数据
    $output = strrev($input) . "\n";
    socket_write($spawn, $output, strlen ($output)) or die("Could not write
    output\n");
    ```
9. 一旦输出被返回到客户端,父/子socket都应通过socket_close()函数来终止
    ```php
    // 关闭 sockets
    socket_close($spawn);
    socket_close($socket);
    
    ```
> 原文链接 [(http://www.cnblogs.com/thinksasa/archive/2013/02/26/2934206.html)](http://www.cnblogs.com/thinksasa/archive/2013/02/26/2934206.html)
        