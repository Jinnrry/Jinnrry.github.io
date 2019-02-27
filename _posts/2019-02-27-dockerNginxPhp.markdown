---
layout:     post
title:      "使用Docker搭建nginx&php环境"
subtitle:   " Docker"
date:       2019-02-27 12:00:00
author:     "木木的木头"
header-img: "img/7.jpg"
catalog: true
tags:
    - Docker
---
> 引言

本来在Linux下使用apt或者yum搭建环境非常快，但是在分布式架构下使用docker部署更方便各个服务的管理。

## 安装php-fpm
```
#拉取镜像
docker pull bitnami/php-fpm
#创建实例
docker run \
  -d \
  -v /data/wwwroot:/usr/share/nginx/html \
  --name m_phpfpm bitnami/php-fpm
```


参数说明：

-d 让容器在后台运行

-v 添加目录映射（将你本地目录映射到容器内）

–-name 容器的名字，随便取，但是必须唯一


## 安装nginx

```
#拉取镜像
docker pull nginx
#创建实例
docker run \
  -d \
  -p 80:80 \
  -v /data/wwwroot:/usr/share/nginx/html \
  -v /data/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /data/nginx/conf:/etc/nginx/conf.d \
  -v /data/wwwlogs:/var/log/nginx \
  --link m_phpfpm:phpfpm \
  --name m_nginx nginx:latest
```

参数说明：

-d 让容器在后台运行

-p 添加主机到容器的端口映射

-v 添加目录映射,这里最好nginx容器的根目录最好写成和php容器中根目录一样。但是不一定非要一模一样,如果不一样在配置nginx的时候需要注意

-–name 容器的名字

–-link 与另外一个容器建立起联系，通过link的方式创建容器，可以使用被Link容器的别名进行访问，而不是通过IP，解除了对IP的依赖。

修改nginx配置文件：

```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        root           html;
        fastcgi_pass   phpfpm:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME /usr/share/nginx/html/$fastcgi_script_name;  #这里一定要写绝对路径，不然会找不到文件
        include        fastcgi_params;
    }

}


```


## Other

如果你配置的时候不用link，用下面命令获取容器ip，然后填进去也可以
```
docker inspect 容器名或ID | grep "IPAddress"
```