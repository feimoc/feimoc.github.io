---
layout:     post
title:      php webdriver
subtitle:   php 进阶
date:       2017-11-16
author:     feimo
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - PHP
    - Webdriver
    - Web爬虫
---
# php webdriver是什么？

Php-webdriver库是Selenium WebDriver的PHP语言绑定，允许您从PHP控制Web浏览器进行自动化测试和web爬虫，支持IE chrome Firefox
个库的概念非常类似于Selenium项目的“官方”Java，.NET，Python和Ruby绑定 
# 安装
  使用Composer可以进行安装。
```$xslt
curl -sS https://getcomposer.org/installer | php
```
然后安装库
```$xslt
php composer.phar require facebook/webdriver
```
# 入门
 我们通过webdriver打开浏览器还需要客户端服务器[selenium](http://selenium-release.storage.googleapis.com/index.html)的支持。<br/>
 下载并运行该文件，用当前的服务器版本替换＃。请记住，您必须安装Java 8+才能启动此命令
 ```$xslt
java -jar selenium-server-standalone-#.jar
 ```
#### 启动Firefox
  确保安装了最新的Firefox和Geckodriver。
  ```$xslt
  $driver  =  RemoteWebDriver :: create($host，DesiredCapabilities :: firefox());
  ```
#### 启动Chrome
   ```$xslt
   $driver  =  RemoteWebDriver :: create($host，DesiredCapabilities :: chrome());
   ```
#### 您也可以自定义所需的功能 例如我在Firefox打开https链接
 ```php
    $host = 'http://localhost:4444/wd/hub';
    $profile = new FirefoxProfile();
    $profile->setPreference('security.enterprise_roots.enabled', true);//设置下profile参数可以正常访问https
    $caps = DesiredCapabilities::firefox();
    $caps->setCapability(FirefoxDriver::PROFILE, $profile);
    $driver = RemoteWebDriver::create($host, $caps);
```
> 参考
> 
> -  [php-webdriver](https://github.com/facebook/php-webdriver)
> - [popzhangzhi/WDsystem](https://github.com/popzhangzhi/WDsystem)