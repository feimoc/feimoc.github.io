---
layout:     post
title:      phpstorm+xdebug
subtitle:   phpstorm
date:       2018-1-4
author:     feimo
header-img: img/post-bg-phpstorm.jpg
catalog: true
tags:
    - phpstorm
    - Xdebug
---

### 环境
- 系统版本 Macos
- PHP版本7.3
- phpstorm版本2020.1

### Xdebug配置
1. 首先去xdebug官网下载对应php版本的xdebug扩展  [链接地址](https://xdebug.org/wizard)
   将phpinfo的信息复制到下图的输入框中，点击here就会出现对应的xdebug扩展。
   ![](http://www.feimoc.com/img/xdebug.png)
2. 打开php.ini添加如下代码
```
[XDebug]
zend_extension =xdebug.so           #xdebug扩展文件 windows是.dll结尾
xdebug.remote_log = "/usr/local/php/xdebug.log"   #开启xdebug日志
xdebug.remote_enable=1            #开启远程调试
xdebug.remote_autostart = 1      #开启远程调试自动启动
xdebug.auto_trace =1             #启用代码自动跟踪
xdebug.collect_vars=1            #收集变量
xdebug.show_error_trace=1        #显示错误信息
xdebug.remote_handler = "dbgp"  #远程调试参数，不需要可以关闭
xdebug.remote_host= "localhost" #本地调试填写localhost 远程调试填写本机真实ip地址
xdebug.remote_port = 9002      #端口号
xdebug.idekey = "PHPSTORM" 
```

### phpstorm配置
#### 指定本地php环境
![](http://www.feimoc.com/img/xdebug01.png)
![](http://www.feimoc.com/img/xdebug02.png)
这里的端口设置和php.ini设置要一致
![](http://www.feimoc.com/img/xdebug03.png)

#### 添加本地虚拟主机域名
![](http://www.feimoc.com/img/xdebug04.png)

### 配置chrome
（如果开启了xdebug.remote_autostart =1 本地调试就不需要安装此插件了）。首先我们需要对浏览器安装Xdebug helper插件，用于在请求中添加参数，类似：XDEBUG_SESSION_START=session_name。
- Chrome: [https://chrome.google.com/webstore/detail/xdebug-helper/](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc)
- Firefox: [https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/](https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/ "Firefox")
![](http://www.feimoc.com/img/xdebug05.png)
更改配置
![](http://www.feimoc.com/img/xdebug06.png)

### 运行项目
#### 运行项目之前开启监听
![](http://www.feimoc.com/img/xdebug07.png)
#### 打断点 浏览器访问
![](http://www.feimoc.com/img/xdebug08.png)
#### 断点成功
![](http://www.feimoc.com/img/xdebug09.png)

- 左侧绿色三角形 ： Resume Program，表示將继续执行，直到下一个中断点停止。
- 左侧红色方形 ： Stop，表示中断当前程序调试。
- 上方第一个图形示 ： Step Over，跳过当前函数。
- 上方第二个图形示 ： Step Into，进入当前函数內部的程序（相当于观察程序一步一步执行）。
- 上方第三个图形示 ： Force Step Into，強制进入当前函数內部的程序。
- 上方第四个图形示 ： Step Out，跳出当前函数內部的程式。
- 上方第五个图形示 ： Run to Cursor，定位到当前光标。
- Variables ：可以观察到所有全局变量、当前局部变量的数值
- Watches ： 可以新增变量，观察变量随着程序执行的变化。
