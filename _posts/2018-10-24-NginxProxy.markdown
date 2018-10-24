---
layout:     post
title:      "Nginx反向代理Github Page项目配置文件"
subtitle:   " Nginx反向代理"
date:       2018-10-24 12:00:00
author:     "蒋为"
header-img: "img/33.jpg"
catalog: true
tags:
    - Nginx
---
```
server {
        listen   80; ## 监听 IPv4 80 端口
        server_name domain.com;

         location / {
                proxy_pass https://jiangwei1995910.github.io;
                proxy_redirect     off;
                proxy_set_header   Host             jiangwei1995910.github.io; ##这一行最重要
                proxy_set_header   X-Real-IP        $remote_addr;
                proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        }
}
```