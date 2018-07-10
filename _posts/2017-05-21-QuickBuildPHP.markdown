---
layout:     post
title:      "ubuntu下php环境快速搭建"
subtitle:   "php环境搭建"
date:       2017-05-21 12:00:00
author:     "蒋为"
header-img: "img/29.jpg"
catalog: true
tags:
    - php
    - linux
---
>记录

1.安装Apache2,打开终端输入命令：sudo apt-get install apache2   回车，输入Y回车 

输入：sudo /etc/init.d/apache2 restart ，回车重启apache,这是为了启动apache


2.安装php

终端输入：sudo apt-get install libapache2-mod-php7.0 php7.0 回车，输入Y回车

输入sudo /etc/init.d/apache2 restart   回车，重启一下apache

输入sudo gedit /var/www/html/test.php,回车，在apache的站根目录下新建一个.php的测试页

在测试页里输入“<?php phpinfo(); ?>”


3.安装mysql数据库

在终端输入：sudo apt-get install mysql-server mysql-client

输入mysql数据库管理员root的登陆密码


4.安装phpmyadmin

到phpmyadmin官网下载个phpmyadmin的压缩包,然后将phpmyadmin解压到网站根目录里面, 

在浏览器输入phpmyadmin的地址，发现php需要安装mbsting

在终端输入：sudo apt-get install php-mbstring,回车Y安装

mbstring安装完后，打开php.ini也就是php的配置文件，将extension=php_mbstring.dll前面的分号去掉保存退出，

php.ini文件在 etc/php/7.0/apache2里，您可以在终端输入：sudo vim /etc/php/7.0/apache2/php.ini直接打开此文件编辑。

再一次输入：sudo /etc/init.d/apache2 restart 重启apache，重新加载php

