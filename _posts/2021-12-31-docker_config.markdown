---
layout:     post
title:      "修改运行中的Docker容器修改端口"
subtitle:   " Docker"
date:       2021-12-31 12:00:00
author:     "Jinnrry"
header-img: "img/6.jpg"
catalog: true
tags:
    - Docker
---

在docker run创建并运行容器的时候，可以通过-p指定端口映射规则。但是，我们经常会遇到刚开始忘记设置端口映射或者设置错了需要修改。当docker start运行容器后并没有提供一个-p选项或设置，让你修改指定端口映射规则。这时可以通过暴力手段，直接强行修改docker容器的配置文件，从而修改容器端口。

## 对应Linux操作系统

1、先查询出容器的Id
```
docker inspect [容器id] | grep Id
```

2、修改容器配置文件

```
vim /var/lib/docker/containers/[容器id]/hostconfig.json
```

3、重启docker

```
sudo systemctl restart docker
```


### 对应Mac操作系统

1、先查询出容器的Id
```
docker inspect [容器id] | grep Id
```

2、进入docker使用的Linux虚拟机

```
 docker run -it --privileged --pid=host [镜像名] nsenter -t 1 -m -u -n -i sh
```


3、修改容器配置文件

```
vim /var/lib/docker/containers/[容器id]/hostconfig.json
```

4、重启docker

```
sudo systemctl restart docker
```
