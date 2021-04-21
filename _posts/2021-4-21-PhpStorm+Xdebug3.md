---
layout:     post
title:      PhpStorm+Xdebug3
subtitle:   phpstorm
date:       2021-3-21
author:     feimo
header-img: img/xdebug3-bg.jpg
catalog: true
tags:
- phpstorm
- Xdebug
---

Xdebug3更新了配置参数，跟Xdebug2配置有所不同，所以需要我们重新配置下。

### 环境
- 系统版本：Macos
- PHP版本 ：7.2
- phpstorm版本：2021.1
- Xdebug版本：3.0.4

### Xdebug配置
1. 首先去 [xdebug](https://xdebug.org/wizard) 官网下载对应php版本的xdebug扩展，将phpinfo的信息复制到下图的输入框中，点击here就会出现对应的xdebug扩展。
   ![](http://www.feimoc.com/img/xdebug.png)
2. 打开php.ini添加如下代码
```
[XDebug]
zend_extension = xdebug.so
xdebug.log  = "/usr/local/php/xdebug.log"  
xdebug.mode = develop,debug
xdebug.start_with_request = default|default
xdebug.client_port = 9003
xdebug.client_host = 127.0.0.1 
xdebug.remote_handler = dbgp 
xdebug.idekey = PHPSTORM
xdebug.cli_color = 2
xdebug.var_display_max_depth = 15
xdebug.var_display_max_data  = 2048
```
在Xdebug 2中，每个功能都有一个启用设置，使用Xdebug 3我们只需要设置xdebug.mode一个参数就行。详细说明请看[官方文档](https://xdebug.org/docs/upgrade_guide)

#### 参数详解
- xdebug.mode 必须与xdebug.start_with_request搭配使用。 不同的mode的不同的用途，如果要多个模式一起开启，就用`,`分隔开就行。develop主要是开启var_dump格式化显示，debug主要是开启步骤调试。详情请参考[官方文档](https://xdebug.org/docs/all_settings#mode)
- xdebug.start_with_request 用于设置xdebug.mode 不同model的启用和关闭
- xdebug.remote_host 、xdebug.remote_port 更改为xdebug.client_host、xdebug.client_port

### phpstorm本地调试配置
#### 指定本地php环境
![](http://www.feimoc.com/img/xdebug3/xdebug1.png)
这里的端口设置和php.ini设置要一致
![](http://www.feimoc.com/img/xdebug3/xdebug2.png)
#### 添加本地虚拟主机域名
![](http://www.feimoc.com/img/xdebug3/xdebug3.png)
### 运行项目
#### 运行项目之前开启监听
![](http://www.feimoc.com/img/xdebug3/xdebug04.png)
#### 打断点 浏览器访问
![](http://www.feimoc.com/img/xdebug3/xdebug05.png)
#### 断点成功
![](http://www.feimoc.com/img/xdebug3/xdebug06.png)
### phpstorm远程调试配置
#### 连接远程服务器
远程调试需要我们配置连接远程服务器配置
![](http://www.feimoc.com/img/xdebug3/remote1.png)
debug server需要勾选use path mapping，其他配置和本地调试一样。
![](http://www.feimoc.com/img/xdebug3/remote2.png)
#### 内网穿透工具（如果是内网和本地虚拟机不需要配置）
由于本地需要和远程服务器通信，我们需要安装内网穿透工具。这里推荐[frp](https://github.com/fatedier/frp)

##### 远程服务器frp配置
```ini
[common]
bind_addr = 0.0.0.0
bind_port = 7000
dashboard_port = 7500
dashboard_user = 设置你的账号
dashboard_pwd = 设置你的密码
privilege_token = frp
```

##### 本地frp配置
```ini
[common]
server_addr = 远程服务器ip
server_port = 7000
privilege_token = frp
log_file = ./frpc.log 
log_level = trace 
log_max_days = 3 


[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 9003
use_encryption = false
use_compression = false
custom_domains = 远程服务器ip
remote_port = 9003
```
#### 配置chrome插件
首先我们需要对浏览器安装Xdebug helper插件，用于在请求中添加参数，类似：XDEBUG_SESSION_START=session_name。
- Chrome: [https://chrome.google.com/webstore/detail/xdebug-helper/](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc)
- Firefox: [https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/](https://addons.mozilla.org/en-US/firefox/addon/the-easiest-xdebug/ "Firefox")
  ![](http://www.feimoc.com/img/xdebug3/remote3.png)
更改配置
  ![](http://www.feimoc.com/img/xdebug3/remote4.png)
#### 开启调试（步骤和本地调试一样）
#####调试成功
![](http://www.feimoc.com/img/xdebug3/remote5.png)
