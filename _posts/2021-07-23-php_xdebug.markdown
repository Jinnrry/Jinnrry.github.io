---
layout:     post
title:      "PHP远程断点配置"
subtitle:   " xdebug配置"
date:       2021-07-23 12:00:00
author:     "Jinnrry"
header-img: "img/26.jpg"
catalog: true
tags:
    - PHP
---
## PHP远程xdebug调试

### 一、安装xdebug

略

### 二、配置xdebug参数

```
[Xdebug]
zend_extension=/home/php7/lib/php/extensions/no-debug-non-zts-20151012/xdebug.so
xdebug.remote_connect_back = 1  
xdebug.remote_enable=1
xdebug.remote_port = 9001
xdebug.remote_handler = dbgp
xdebug.auto_trace = 1
xdebug.remote_log = /tmp/xdebug.log
xdebug.remote_autostart=1  # 全部请求都会开启断点调试

```

### 三、PHPSTORM配置

1、在设置中找到PHP->Debug选项卡，然后将Xdebug里面的debug port改成9001

2、PHP->Servers选项卡中，新建一个你需要调试的server，配置服务端代码和本地代码的目录映射