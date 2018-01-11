---
layout:     post
title:      Swoole深入学习(2)Process
subtitle:   php 进阶
date:       2018-1-11
author:     feimo
header-img: img/tag-bg-o.jpg
catalog: true
tags:
    - PHP
    - Swoole
---
swoole-1.7.2增加了一个进程管理模块，用来替代PHP的pcntl扩展。pcntl是php新增的一个多进程扩展，用来实现多进程，但是有很多不完善的地方，swoole 就完善了这些地方，而且使得使用非常简单。
### 创建一个多进程
swoole创建多进程很简单：new Swoole\Process('callback_function') 就可以了。

比如我要同时创建10个进程，就for 循环10次就可以了。
```php
for($i=0; $i<=10 ; $i++){
    $process = new Swoole\Process('callback_function');
    $pid = $process->start();
    echo PHP_EOL . $pid;//
}
```


### 进程间的通信
如果是非常简单的多进程执行任务，那么进程间就不需要通讯了，实际情况下，很多业务是需要通讯的，比如，发邮件，如果子进程发送失败了，那么是要通知主进程的等等。

再看进程间通信之前，先看下 Swoole\Process 的几个参数：
```php
Swoole\Process(mixed $function, $redirect_stdin_stdout = false, $create_pipe = true);
```
它有三个参数：
- $function：子进程创建成功后要执行的函数

- $redirect_stdin_stdout：重定向子进程的标准输入和输出。 设置为true，则在进程内echo将不是打印屏幕，而是写入到管道，读取键盘输入将变为从管道中读取数据。 默认为false，阻塞读取。

- $create_pipe：是否创建管道，启用$redirect_stdin_stdout后，此选项将忽略用户参数，强制为true 如果子进程内没有进程间通信，可以设置为false。
 
swoole_process进程间支持2种通信方式
- 管道pipe
- 消息队列

#### 管道通信
管道通信是swoole_process默认的一种通信方式。当然我们也可以在实例化的时候通过参数来设定：
```php
$process = new Swoole\Process('callback_function', false, true);
```
这样就创建了一个管道通信进程。我们打印下$process这个对象的值：
```php
var_dump($process)
object(swoole_process)#1 (3) {
  ["pipe"]=>
  int(2)
  ["callback"]=>
  string(26) "callback_function"
  ["pid"]=>
  int(4333)
}
```
里面有个字段pipe是管道id，还有一个pid是进程id，所以：
每次创建一个进程后，就会随之创建一个管道，主进程想和哪一个进程通信，就向那个进程的管道写入/读取数据。
管道有2个方法，分别来写入数据，和读取数据:

```
write()  和  read() 
```
来个例子，看下如何通过管道通信

```php
<?php
/**
 * Created by PhpStorm.
 * User: feimo
 * Date: 2018\1\11 
 * Time: 14:41
 */

//进程数量
$worker_num = 2;
$workers = [];
for ($i = 0; $i < $worker_num; $i++) {
    $process = new Swoole\Process('callback_function', false);
    $pid = $process->start();
    //将每一个进程的句柄存起来
    $workers[$pid] = $process;
}
// 主进程，通过管道给子进程发送数据
foreach ($workers as $pid => $process) {
    //向子进程管道里写内容：$process->write($data);
    $process->write("hello worker[$pid]\n");
    //从子进程管道里面读取信息：$process->read();
    echo "From Worker: ".$process->read();
}
//子进程执行的回调函数
function callback_function($worker){
    //从主进程管道中读取
    $recv = $worker->read();
    echo PHP_EOL. "From Master: $recv\n";
    //向主进程管道中写入数据
    $worker->write("hello master , this pipe  is ". $worker->pipe .";  this  pid  is ".$worker->pid."\n");
    $worker->exit(0);
}
```
运行看下效果：
```shell
$ php process_pipe.php
From Master: hello worker[6759]
From Worker: hello master , this pipe  is 4;  this  pid  is 6759
From Master: hello worker[6760]
From Worker: hello master , this pipe  is 6;  this  pid  is 6760
```
注意 主进程和子进程中的 read() 不能一开始都读，得写反。不然管道中没数据，就会阻塞住了。

所以可以参考下面的图：
![](https://i.imgur.com/hAhgdgb.png)
所以一般的顺序是：
```
master-->write()

work-->read()

work-->write()

master-->read()
```
这样才能有序的使用通道，才不会被阻塞。而且是一对一的，write 2 次，也要read 2次，先write先read。

第二个参数 $redirect_stdin_stdout 说，设置为 true ，子进程会将 echo 写入到主管道。我把上面的代码改一下，看下输出结果是啥。

```php
$process = new Swoole\Process('callback_function', true, true);
```
紧紧改动了这一行，再看下：

```php
//进程数量
$worker_num = 2;
$workers = [];
for ($i = 0; $i < $worker_num; $i++) {
    $process = new Swoole\Process('callback_function', true);
    $pid = $process->start();
    //将每一个进程的句柄存起来
    $workers[$pid] = $process;
}
// 主进程，通过管道给子进程发送数据 
foreach ($workers as $pid => $process) {
    //向子进程管道里写内容：$process->write($data);
    $process->write("hello worker[$pid]\n");
    //从子进程管道里面读取信息：$process->read();
    echo "From Worker: ".$process->read();
}
//子进程执行的回调函数
function callback_function($worker){
    //从主进程管道中读取    
    $recv = $worker->read();
    //这个echo 相当于在往master管道里写数据。write('From Master: hello worker[9251]')
    echo "From Master: $recv\n";
    //第二次写, 但是主进程没有第二次read()，所以没有被读到。
    $worker->write("hello master , this pipe  is ". $worker->pipe .";  this  pid  is ".$worker->pid."\n");
    $worker->exit(0);
}
```
运行下输出结果为：

```shell
$ php test.php
From Worker: From Master: hello worker[9251]
From Worker: From Master: hello worker[9252]
```
来分析下，因为$redirect_stdin_stdout为true，所以子进程中echo的内容就到了主管道里面，而不是打印在屏幕上，所以，主进程从管道里读到的内容，就是子进程中echo的内容。 
也就造成了上面的输出结果。

那么如何使子进程中的第二个write，能被主进程读到呢？很简单，在主进程中在 read() 一次就可以了：
```php
// 主进程，通过管道给子进程发送数据 
foreach ($workers as $pid => $process) {
    //向子进程管道里写内容：$process->write($data);
    $process->write("hello worker[$pid]\n");
    //从子进程管道里面读取信息：$process->read();
    echo "From Worker: ".$process->read();
    //第二次读
    echo "From Worker: ".$process->read();
}
```
再看下打印结果：

```shell
From Worker: From Master: hello worker[9328]
From Worker: hello master , this pipe  is 4;  this  pid  is 9328
From Worker: From Master: hello worker[9329]
From Worker: hello master , this pipe  is 6;  this  pid  is 9329
```
#### 消息队列
swoole进程通信还有第二种方式就是“消息队列”，这个消息队列其实就是Linux系统里面的msgqueue。

swoole提供了2个方法，来实现消息队列的通信。

```
pop() 和 push() 
```
要使用队列，必须在start方法前使用useQueue。

```php
<?php
$workers = [];
$worker_num = 2;
for($i = 0; $i < $worker_num; $i++)
{
    $process = new Swoole\Process('callback_function',false,false);
    $process->useQueue();
    $pid = $process->start();
    $workers[$pid] = $process;
}
foreach($workers as $pid => $process)
{
    //给子进程发布消息
    $process->push("hello worker[$pid]\n");
}
function callback_function($worker)
{
    //接受来自主进程的消息
    $recv = $worker->pop();
    echo "From Master: $recv\n";
    $worker->exit(0);
}
//等待消息停止
for($i = 0; $i < $worker_num; $i++)
{
    $ret = Swoole\Process::wait();
    $pid = $ret['pid'];
    unset($workers[$pid]);
    echo "Worker Exit, PID=".$pid.PHP_EOL;
}
```
运行下：

```shell
$ php process_msg.php
From Master: hello worker[18219]
From Master: hello worker[18218]
Worker Exit, PID=18218
Worker Exit, PID=18219
```
消息队列，依赖于linux系统ipcs，所以，如果空间满了，可能会阻塞。或者出现其他一些异常，具体可以参考swoole内核设置：[https://wiki.swoole.com/wiki/page/p-server/sysctl.html](https://wiki.swoole.com/wiki/page/p-server/sysctl.html)

与消息队列相关的几个系统命令：

查看消息队列
```
ipcs -q
```
ipcrm 删除消息队列

```
ipcrm -q MessageId
```
批量删除所有的队列

```
ipcs -q | sed "$ d; 1,2d" |  awk '{ print "Removing " $2; system("ipcrm -q " $2) }'
```