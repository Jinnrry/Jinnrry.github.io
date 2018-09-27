---
layout:     post
title:      "在apache2中部署Django项目"
subtitle:   " Linux"
date:       2018-09-27 12:00:00
author:     "蒋为"
header-img: "img/30.jpg"
catalog: true
tags:
    - Linux
---
1. 安装 apache2 和 mod_wsgi
```
sudo apt-get install apache2
# Python 2
sudo apt-get install libapache2-mod-wsgi
# Python 3
sudo apt-get install libapache2-mod-wsgi-py3
```
2. 确认安装的apache2版本号
```
apachectl -v
Server version: Apache/2.4.6 (ubuntu)
Server built:   Dec  5 2013 18:32:22
```
3. 准备一个新网站
ubuntu的apache2配置文件在 /etc/apache2/ 下

备注：centos 用户 apache 名称为 httpd 在 /etc/httpd/ 中

新建一个网站配置文件
```
sudo vi /etc/apache2/sites-available/sitename.conf
示例内容如下：

<VirtualHost *:80>
    ServerName www.yourdomain.com
    ServerAlias otherdomain.com
    ServerAdmin tuweizhong@163.com
  
    Alias /media/ /home/tu/blog/ #注意改位置
    Alias /static/ /home/tu/static/ #注意改位置
  
    <Directory /home/tu/blog>
        Require all granted
    </Directory>
  
    <Directory /home/tu/static>
        Require all granted
    </Directory>
  
    WSGIScriptAlias / /home/tu/blog/wsgi.py #注意改位置
    # WSGIDaemonProcess ziqiangxuetang.com python-path=/home/tu/blog:/home/tu/.virtualenvs/blog/lib/python2.7/site-packages
    # WSGIProcessGroup ziqiangxuetang.com
  
    <Directory /home/tu/blog/blog>
    <Files wsgi.py>
        Require all granted
    </Files>
    </Directory>
</VirtualHost>
```
如果你的apache版本号是 2.2.x（第二步有方法判断）

用下面的代替  Require all granted

Order deny,allow
Allow from all
备注：把上面配置文件中这两句的备注去掉，可以使用 virtualenv 来部署网站，当然也可以只写一个 /home/tu/blog

    # WSGIDaemonProcess ziqiangxuetang.com python-path=/home/tu/blog:/home/tu/.virtualenvs/blog/lib/python2.7/site-packages
    # WSGIProcessGroup ziqiangxuetang.com


4. 修改wsgi.py文件
注意：上面如果写了 WSGIDaemonProcess 的话，这一步可以跳过，即可以不修改 wsgi.py 文件。



上面的配置中写的 WSGIScriptAlias / /home/tu/blog/blog/wsgi.py

就是把apache2和你的网站project联系起来了
```
import os

import sys

sys.path.append("/home/tu/blog/")    #这里修改为项目路径，wsgi.py的上一级路径
from django.core.wsgi import get_wsgi_application

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "fxdl.settings")

application = get_wsgi_application()
```

第 3，4，5 行为新加的内容，作用是让脚本找到django项目的位置，也可以在sitename.conf中做，用WSGIPythonPath,想了解的自行搜索, 第 7 行如果一台服务器有多个django project时一定要修改成上面那样，否则访问的时候会发生网站互相串的情况，即访问A网站到了B网站，一会儿正常，一会儿又不正常（当然也可以使用 mod_wsgi daemon 模式,点击这里查看）

激活网站
```
sudo a2ensite sitename 或 sudo a2ensite sitename.conf
```

自此，部署完成，如果网站没有跑起来，可以到/var/log/apache2/error.log中查看错误信息，根据错误信息修改即可。一般都会有错，比如python包缺失，文件夹权限。。。。


