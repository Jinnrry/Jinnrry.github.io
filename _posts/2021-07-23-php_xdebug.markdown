---
layout:     post
title:      "PHP远程断点配置"
subtitle:   " xdebug配置"
date:       2021-07-23 12:00:00
author:     "Jinnrry"
header-img: "img/27.jpg"
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

1、在设置中找到PHP->Debug选项卡，然后将Xdebug里面的debug port改成9001,然后点击start listenint，开启监听

2、PHP->Servers选项卡中，新建一个你需要调试的server，配置服务端代码和本地代码的目录映射


### 四、修改php、nginx超时时间

因为是一个http请求，所以就涉及到nginx和php-fpm，这两个都不能超时，php-fpm自己不能超时，nginx等待php-fpm也不可以，这两个都要设置

nginx：

```
keepalive_timeout 3600;
#tcp_nodelay on;
fastcgi_connect_timeout 3600;
fastcgi_send_timeout 3600;
fastcgi_read_timeout 3600;
```

php-fpm:

```
request_terminate_timeout = 3600s
```

php.ini

```
max_execution_time = 0
```
