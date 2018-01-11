---
layout:     post
title:      Swoole深入学习(1)tcp Server和tcp Client
subtitle:   php 进阶
date:       2018-1-11
author:     feimo
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - PHP
    - Swoole
---
php是单进程的，没法在一个程序块中使用多进程来处理一个复杂的逻辑，即使后来出现了pcntl_fork模块来处理多进程，但是它比较鸡肋，并不适用于windows平台，其实最大的问题是它使用起来非常之复杂和繁琐，难以理解。

php并不支持异步。所以，在处理一些高并发高负载的请求的时候，我们往往会考虑用nodejs来做。

那么，有没有一种办法，能够让php很好的支持异步、异步、简单的使用tcp/udp/socket服务呢？swoole应运而生了！！！
### swoole 简介
[swoole](http://www.swoole.com/ )是PHP的异步、并行、高性能网络通信引擎，使用纯C语言编写，提供了PHP语言的异步多线程服务器，异步TCP/UDP网络客户端，异步MySQL，异步Redis，数据库连接池，AsyncTask，消息队列，毫秒定时器，异步文件读写，异步DNS查询。 Swoole内置了Http/WebSocket服务器端/客户端、Http2.0服务器端。

接下来通过几个例子来介绍下swoole的tcpServer和tcpClient

### tcpServer和tcpClient
#### server
```php
<?php

/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11 
 * Time: 13:05
 */
class Server
{
    private $serv;

    public function __construct()
    {
        $this->serv = new Swoole\Server('127.0.0.1', 9501);
        //当启动一个Swoole应用时，一共会创建2 + n + m个进程，2为一个Master进程和一个Manager进程，其中n为Worker进程数。m为TaskWorker进程数。
        //默认如果不设置，swoole底层会根据当前机器有多少CPU核数，启动对应数量的Reactor线程和Worker进程。我机器为4核的。Worker为4。TaskWorker为0。
        //下面我来设置worker_num = 10。看下启动了多少个进程
        $this->serv->set([
            'worker_num' => 10,
            //'task_worker_num' => 2,
            'deamonize' => true,
        ]);

        //启动10个work，总共12个进程。
        /*
        ➜  Event git:(master) pstree |grep server.php
    |   \-+= 54172  php server.php  #Master进程
    |     \-+- 54173  php server.php  # Manager 进程
    |       |--- 54174  php server.php  #Work 进程
    |       |--- 54175  php server.php
    |       |--- 54176  php server.php
    |       |--- 54177  php server.php
    |       |--- 54178  php server.php
    |       |--- 54179  php server.php
    |       |--- 54180  php server.php
    |       |--- 54181  php server.php
    |       |--- 54182  php server.php
    |       \--- 54183  php server.php
         *
         */

        //增加新的监控的ip:post:mode
        $this->serv->addlistener("::1", 9500, SWOOLE_SOCK_TCP);

        //监听事件
        /*
         *
         * - onStart
         * - onShutdown
         * - onWorkerStart
         * - onWorkerStop
         * - onTimer
         * - onConnect
         * - onReceive
         * - onClose
         * - onTask
         * - onFinish
         * - onPipeMessage
         * - onWorkerError
         * - onManagerStart
         * - onManagerStop
         */

        $this->serv->on('Start', array($this, 'onStart'));
        $this->serv->on('Connect', array($this, 'onConnect'));
        $this->serv->on('Receive', array($this, 'onReceive'));
        $this->serv->on('Close', array($this, 'onClose'));

        //master进程启动后, fork出Manager进程, 然后触发ManagerStart
        $this->serv->on('ManagerStart', function (\swoole_server $server) {
            echo "On manager start.\n";
        });

        //manager进程启动,启动work进程的时候调用 workid表示第几个id, 从0开始。
        $this->serv->on('WorkerStart', function ($serv, $workerId) {
            echo $workerId . "---\n";
        });

        //当一个work进程死掉后，会触发
        $this->serv->on('WorkerStop', function () {
            echo "--stop\n";
        });

        //启动
        $this->serv->start();
    }

    //启动server时候会触发。
    public function onStart($serv)
    {
        echo "Start\n";
    }

    //client连接成功后触发。
    public function onConnect($serv, $fd, $from_id)
    {
        $a = $serv->send($fd, "Hello {$fd}!");

    }

    //接收client发过来的请求
    public function onReceive(swoole_server $serv, $fd, $from_id, $data)
    {
        echo "Get Message From Client {$fd}:{$data}\n";
        $serv->send($fd, $data);

    }


//客户端断开触发
    public function onClose($serv, $fd, $from_id)
    {
        echo "Client {$fd} close connection\n";


    }


}


// 启动服务器
$server = new Server();
```
启动服务端server：
```linux
php server.php
On manager start.
0---
1---
2---
3---
4---
5---
6---
7---
8---
9---
```
我们来分析整个server 启动的步骤：
- 启动php server.php后，当前进程fork出Master进程，然后退出。
- Master进程启动成功之后，fork出Manager进程，并触发OnManagerStart事件。
- Manager进程启动成功时候，fork出Worker进程，并触发OnWorkerStart事件。

#### syncClient
server端好了，那么就会需要client端来连接，swoole里面client分为同步和异步，先来一个同步clent客户端。
```php
<?php
/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11
 * Time: 13:29
 */


// sync 同步客户端
class client
{
    private $client;

    public function __construct()
    {
        $this->client = new Swoole\Client(SWOOLE_SOCK_TCP | SWOOLE_KEEP);
        $this->client->connect('127.0.0.1', 9501, 1);
    }

    public function connect()
    {
        //fwrite(STDOUT, "请输入消息：");
        //$msg = trim(fgets(STDIN));
        $msg = rand(1,12);

        //发送给消息到服务端
        $this->client->send( $msg );

        //接受服务端发来的信息
        $message = $this->client->recv();
        echo "Get Message From Server:{$message}\n";

        //关闭客户端
        $this->client->close();

    }
}
$client = new Client();
$client->connect();
```
同步client是同步阻塞的，一整套connect->send()->rev()->close()是同步进行的。所以，如果是大量的循环数据，就不适合同步client了。
比如下面： 
```php
<?php
/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11
 * Time: 13:29
 */


// sync 同步客户端
class syncClient
{
    private $client;

    public function __construct()
    {
        var_dump(swoole_get_local_ip());
        $this->client = new Swoole\Client(SWOOLE_SOCK_TCP | SWOOLE_KEEP);
        $this->client->connect('127.0.0.1', 9501, 1);

        $i = 0;
        while ($i < 100) {
            $this->client->send($i."\n");
            $message = $this->client->recv();
            echo "Get Message From Server:{$message}\n";
            $i++;
        }
    }
}
$client = new syncClient();
```
### asyncClient
```php
<?php
/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11
 * Time: 13:32
 */

//异步客户端

$client = new Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
$client->on("connect", function($cli) {

    var_dump($cli->isConnected()); // true
    var_dump($cli->getsockname()); //['port' => 57305, 'host'=> '127.0.0.1']
    var_dump($cli->sock); // 5

    $i = 0;
    while ($i < 100) {
        $cli->send($i."\n");
        $i++;
    }
    //关闭
    //$cli->close();
});

$client->on("receive", function($cli, $data){
    echo "Receive: $data";
});

$client->on("error", function(swoole_client $cli){
    echo "error\n" . $cli->errCode;
});

$client->on("close", function(swoole_client $cli){
    echo "Connection close\n";
});

$client->connect('127.0.0.1', 9501);
```
这样就是一个异步的client了，处理更快，但是只支持php的cli模式。
#### server与client交互
1. Client主动Connect的时候，Client实际上是与Master进程中的某个Reactor线程发生了连接。
2. 当TCP的三次握手成功了以后，由这个Reactor线程将连接成功的消息告诉Manager进程，再由Manager进程转交给Worker进程。
3. 在这个Worker进程中触发了OnConnect的方法。
4. 当Client向Server发送了一个数据包的时候，首先收到数据包的是Reactor线程，同时Reactor线程会完成组包，再将组好的包交给Manager进程，由Manager进程转交给Worker。
5. 此时Worker进程触发OnReceive事件。
6. 如果在Worker进程中做了什么处理，然后再用Send方法将数据发回给客户端时，数据则会沿着这个路径逆流而上。

关于上面说到的几个进程，解释下：
Master进程是一个多线程进程，其中有一组非常重要的线程，叫做Reactor线程（组），每当一个客户端连接上服务器的时候，都会由Master进程从已有的Reactor线程中，根据一定规则挑选一个，专门负责向这个客户端提供维持链接、处理网络IO与收发数据等服务。

而Manager进程，某种意义上可以看做一个代理层，它本身并不直接处理业务，其主要工作是将Master进程中收到的数据转交给Worker进程，或者将Worker进程中希望发给客户端的数据转交给Master进程进行发送。另外，Manager进程还负责监控Worker进程，如果Worker进程因为某些意外挂了，Manager进程会重新拉起新的Worker进程，有点像Supervisor的工作。

Worker进程了，顾名思义，Worker进程其实就是处理各种业务工作的进程，Manager将数据包转交给Worker进程，然后Worker进程进行具体的处理，并根据实际情况将结果反馈给客户端。
#### task_worker
在swoole中work进程分为EventWorker和TaskWorker，对应的配置文件设置为：
```php
$this->serv->set([
    'worker_num' => 10,  #EventWorker
    'task_worker_num' => 2,  #TaskWorker
    'deamonize' => true,
]);
```
worker是基于event触发，而task则是manager直接生成的子进程。那么他们有什么区别呢？

共同点是：他们都是最底层负责处理业务的进程。

Swoole的业务逻辑部分是同步阻塞运行的，如果遇到一些耗时较大的操作，例如访问数据库、广播消息等，就会影响服务器的响应速度。因此Swoole提供了Task功能，将这些耗时操作放到另外的进程去处理，当前woker进程继续执行后面的逻辑。运行Task,需要在swoole服务中配置参数task_worker_num,即可开启task功能。此外，必须给swoole_server绑定两个回调函数：onTask和onFinish。这两个回调函数分别用于执行Task任务和处理Task任务的返回结果。

先来写一个demo，来如何用 taskWoker来处理业务
#### taskServer
```php
<?php

/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11
 * Time: 13:39
 */
class taskServer
{
    private $serv;

    /**
     * [__construct description]
     * 构造方法中,初始化 $serv 服务
     */
    public function __construct()
    {
        $this->serv = new Swoole\Server('0.0.0.0', 9501);
        //初始化swoole服务
        $this->serv->set(array(
            'worker_num' => 8,
            'daemonize' => false, //是否作为守护进程,此配置一般配合log_file使用
            'max_request' => 1000,
            'log_file' => './swoole.log',
            'task_worker_num' => 8
        ));

        //设置监听
        $this->serv->on('Start', array($this, 'onStart'));
        $this->serv->on('Connect', array($this, 'onConnect'));
        $this->serv->on("Receive", array($this, 'onReceive'));
        $this->serv->on("Close", array($this, 'onClose'));
        $this->serv->on("Task", array($this, 'onTask'));
        $this->serv->on("Finish", array($this, 'onFinish'));

        //开启
        $this->serv->start();
    }

    public function onStart($serv)
    {
        echo SWOOLE_VERSION . " onStart\n";
    }

    public function onConnect($serv, $fd)
    {
        echo $fd . "Client Connect.\n";
    }

    public function onReceive($serv, $fd, $from_id, $data)
    {
        echo "Get Message From Client {$fd}:{$data}\n";
        // send a task to task worker.
        $param = array(
            'fd' => $fd
        );
        // start a task
        $serv->task(json_encode($param));

        echo "Continue Handle Worker\n";
    }

    public function onClose($serv, $fd)
    {
        echo "Client Close.\n";
    }

    //执行Task任务
    public function onTask($serv, $task_id, $from_id, $data)
    {
        echo "This Task {$task_id} from Worker {$from_id}\n";
        echo "Data: {$data}\n";
        for ($i = 0; $i < 100; $i++) {
            sleep(1);
            echo "Task {$task_id} Handle {$i} times...\n";
        }
        $fd = json_decode($data, true);
        $serv->send($fd['fd'], "Data in Task {$task_id}");
        return "Task {$task_id}'s result";
    }

    //处理Task任务的返回结果
    public function onFinish($serv, $task_id, $data)
    {
        echo "Task {$task_id} finish\n";
        echo "Result: {$data}\n";
    }
}

$server = new taskServer();
```
### taskClient
```php
<?php
/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11
 * Time: 13:46
 */
class taskClient
{
    private $client;

    public function __construct() {
        $this->client = new Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_ASYNC);
        $this->client->on('Connect', array($this, 'onConnect'));
        $this->client->on('Receive', array($this, 'onReceive'));
        $this->client->on('Close', array($this, 'onClose'));
        $this->client->on('Error', array($this, 'onError'));
    }

    public function connect() {
        if(!$fp = $this->client->connect("127.0.0.1", 9501 , 1)) {
            echo "Error: {$fp->errMsg}[{$fp->errCode}]\n";
            return;
        }
    }

    //connect之后,会调用onConnect方法
    public function onConnect($cli) {
        fwrite(STDOUT, "Enter Msg:");
        swoole_event_add(STDIN,function(){
            fwrite(STDOUT, "Enter Msg:");
            $msg = trim(fgets(STDIN));
            $this->send($msg);
        });
    }

    public function onClose($cli) {
        echo "Client close connection\n";
    }

    public function onError() {

    }

    public function onReceive($cli, $data) {
        echo "Received: ".$data."\n";
    }

    public function send($data) {
        $this->client->send($data);
    }

    public function isConnected($cli) {
        return $this->client->isConnected();
    }

}

$client = new taskClient();
$client->connect();
```
运行一下：
```linux
php taskServer.php 
php taskClient.php
```
服务端打印：
```linux
$ php task_server.php
2.0.12 onStart
1Client Connect.
Get Message From Client 1:12345
Continue Handle Worker
This Task 0 from Worker 3
Data: {"fd":1}
Task 0 Handle 0 times...
Task 0 Handle 1 times...
Task 0 finish
Result: Task 0's result
```
客户端打印：
```linux
$ php task_client.php
Enter Msg:12345
Enter Msg:Received: Data in Task 0
```
这里面有几点需要注意： 
1. 运行Task,必须要在swoole服务中配置参数task_worker_num,此外，必须给swoole_server绑定两个回调函数：onTask和onFinish。 
2. onTash 要return 数据 
3. onFinish 会接收到onTash的数据，标记成完成。 
4. swoole_event_add 把输入绑定成事件，这个后续将，这样client就可以连续的多次输入。
#### swoole的架构
上面说了这么，图表总结一下swoole结构：
swoole采用 多线程Reactor+多进程Worker
![](https://i.imgur.com/ZlyUXX0.png)
swoole的处理连接流程图如下：
![](https://i.imgur.com/sZOWxJ0.png)
当请求到达时,swoole是这样处理的：
请求到达 Main Reactor 
| 
| 
Main Reactor根据Reactor的情况，将请求注册给对应的Reactor 
(每个Reactor都有epoll。用来监听客户端的变化) 
| 
| 
客户端有变化时，交给worker来处理 
| 
| 
worker处理完毕，通过进程间通信(比如管道、共享内存、消息队列)发给对应的reactor。 
| 
| 
reactor将响应结果发给相应的连接 
| 
| 
请求处理完成
因为reactor基于epoll，所以每个reactor可以处理无数个连接请求。 如此，swoole就轻松的处理了高并发。
> 原文链接 [(http://blog.csdn.net/think2me/article/details/53696206)](http://blog.csdn.net/think2me/article/details/53696206)
        
