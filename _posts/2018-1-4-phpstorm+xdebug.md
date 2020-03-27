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
- 系统版本 Windows
- PHP版本7.0
- phpstorm版本 2017.1

### Xdebug配置
1. 首先去xdebug官网下载对应php版本的xdebug 放到php ext目录下面：F:\phpStudy\php\php-7.0.12-nts\ext [链接地址](https://xdebug.org/download.php)
2. 打开php.ini添加如下代码
```
[XDebug]
xdebug.profiler_output_dir="F:\phpStudy\tmp\xdebug"
xdebug.trace_output_dir="F:\phpStudy\tmp\xdebug"
xdebug.remote_enable=1  
xdebug.var_display_max_children=128 
xdebug.var_display_max_data=51200000     
xdebug.var_display_max_depth=5  
xdebug.remote_host = "127.0.0.1"
xdebug.remote_port = 9000
xdebug.idekey = PHPSTORM 
zend_extension="F:\phpStudy\php\php-7.0.12-nts\ext\php_xdebug.dll"   
```

### phpstorm配置
#### 指定本地php环境
![](http://www.feimoc.com/img/ip4Ov4g.png)
![](http://www.feimoc.com/img/8rQZQYW.png)
这里的端口设置和php.ini设置要一致，我这里设置为9000如果端口冲突请自行设置
![](http://www.feimoc.com/img/QljWG4i.png)

#### 添加本地虚拟主机域名
![](http://www.feimoc.com/img/lAO1gso.png)

### 配置chrome
首先我们需要对浏览器安装Xdebug helper插件，用于在请求中添加参数，类似：XDEBUG_SESSION_START=session_name。
- Chrome: [https://chrome.google.com/webstore/detail/xdebug-helper/](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc)
- Firefox: [https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/](https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/ "Firefox")
![](http://www.feimoc.com/img/w67ICBB.png)
更改配置
![](http://www.feimoc.com/img/a0p5ob4.png)

### 运行项目
#### 运行项目之前开启监听
![](http://www.feimoc.com/img/PuDxm74.png)
#### 打断点 浏览器访问
![](http://www.feimoc.com/img/HMPhBJH.png)
#### 首次运行域名如果出现一个弹框，点击Accept
![](http://www.feimoc.com/img/IBGbxbu.png)
#### 断点成功
![](http://www.feimoc.com/img/buNulUu.png)

- 左侧绿色三角形 ： Resume Program，表示將继续执行，直到下一个中断点停止。
- 左侧红色方形 ： Stop，表示中断当前程序调试。
- 上方第一个图形示 ： Step Over，跳过当前函数。
- 上方第二个图形示 ： Step Into，进入当前函数內部的程序（相当于观察程序一步一步执行）。
- 上方第三个图形示 ： Force Step Into，強制进入当前函数內部的程序。
- 上方第四个图形示 ： Step Out，跳出当前函数內部的程式。
- 上方第五个图形示 ： Run to Cursor，定位到当前光标。
- Variables ：可以观察到所有全局变量、当前局部变量的数值
- Watches ： 可以新增变量，观察变量随着程序执行的变化。