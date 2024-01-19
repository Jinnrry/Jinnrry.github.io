---
layout:     post
title:      "快速搭建一个属于自己的域名邮箱服务"
subtitle:   "域名邮箱"
date:       2024-01-19 12:00:00
author:     "Jinnrry"
header-img: "img/7.jpg"
catalog: true
tags:
    - email
---
# 前言

为什么要自建邮箱？邮箱就像电话号码一样，是一个联系方式，这个联系方式一旦留给对方就没法改了。因此一个常用邮箱一旦丢失，损失十分巨大。通常来说，我们使用腾讯、网易、Google这些大公司的邮箱服务，一般来说这些公司也不会倒闭，但是！万一呢？现在的互联网巨头，说不定十年二十年后就倒了呢。除了这个风险，更加需要担心的是莫名其妙的封号，比如发了什么敏感内容、或者你没绑定手机号、没实名，都可能成为封号的理由。而自建的话就没这些担忧，真正的把邮箱掌握在自己手上，只要我一直续费域名，那这个邮箱就不可能丢。

除此之外，自建邮箱的话，这个域名下面的所有账号都是你的，比如xiaoming@xx.com lixiaoming@xx.com都可能是你的，往这两个地址发件，你都能收到内容。不仅如此，你甚至可以在每个地方都留下不同的账号，比如腾讯论坛你就留tencent@xx.com，阿里论坛你就留ali@xx.com，你给你妈就留mm@xx.com，给你同学就留tongxue@xx.com。你给他们每个人都是不同的地址，但是最终收件的都是你。

# 准备材料

1、既然是域名邮箱，那当然需要有一个域名啊，阿里云、腾讯云、GoDaddy等地方买一个你喜欢的就行

2、一台服务器，随便哪里买一台最最最低配，最最最便宜的就行。

# 开始搭建

> 为了方便管理，我这里使用Docker搭建，邮件服务端程序使用[PMain](https://github.com/Jinnrry/PMail)，这个服务端是专为个人使用场景设计的，资源占用极低（仅需15Mb磁盘，10Mb内存即可运行），搭建运维方便

## 1、安装Docker

参考[Docker官方文档](https://docs.docker.com/engine/install/debian/)

如果是Debian系操作系统，执行
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

如果是CentOS系操作系统，执行
```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
```

安装完成后执行`docker`命令，看到类似这里的输出就说明安装Docker成功了
```
Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Common Commands:
  run         Create and run a new container from an image
  exec        Execute a command in a running container
  ps          List containers
  build       Build an image from a Dockerfile
  pull        Download an image from a registry
  push        Upload an image to a registry
  images      List images
  login       Log in to a registry
  logout      Log out from a registry
  search      Search Docker Hub for images
  version     Show the Docker version information
  info        Display system-wide information
	(.....省略....)
```

## 2、安装PMail

执行如下命令

```
docker run -p 25:25 -p 80:80 -p 443:443 -p 110:110 -p 465:465 -v $(pwd)/config:/work/config ghcr.io/jinnrry/pmail:latest
```

然后检查是否安装成功，执行
```
docker ps | grep pmail
```

如果看到输出就说明安装成功了

## 3、配置邮箱

1、使用浏览器打开http://[你服务器IP]

2、正常情况下你会看到一个欢迎页面，直接点击下一步即可。

3、数据库选择页面，你啥也不懂的话，直接下一步

4、管理员账号设置页面，输入账号密码即可

5、域名设置页面，输入你的域名，比如我的域名的`jinnrry.com`，SMTP域名那一栏就是`smtp.jinnrry.com`，因为smtp已经存在了，填入`jinnrry.com`即可。下面Web管理后台是你以后网页邮箱地址，填一个你喜欢的就好，比如`email.jinnrry.com`

6、DNS设置，这个地方会给你一个表格，你按照这个表格的内容填入你域名的DNS解析记录即可。
(域名解析记录在你买域名的服务商那里编辑)

7、SSL证书设置，直接自动配置即可，点击下一步

正常情况下，等待几分钟后就能进入邮箱页面了。

## 4、测试邮箱是否正常工作

1、发邮件测试，随便给你以前的哪个邮箱发封邮件试试能不能收到

2、收邮件测试，随便用你以前的哪个邮箱给你的域名邮箱发邮件试试能不能收到。（域名邮箱前缀不重要，随便什么前缀都能收到邮件）

3、邮箱跑分测试，打开[Mail Test](https://www.mail-tester.com/)，首页他会给你显示一个邮箱地址，用你的域名邮箱给这个地址发一封邮件，然后他就会给你打个分。如果不是满分的话按照他的建议改一改即可，正常配置的话都是能够得到满分的。

## 5、数据安全

如果你啥的不懂的话，那么你按时备份你服务器里面的config目录即可，如果你使用其他数据库，也记得备份config目录，这里面有DKIM SSL秘钥，丢失以后重新生成很麻烦

## 6、其他问题

如果你部署失败，或者使用过程中遇到Bug，你可以到[PMail Discussions](https://github.com/Jinnrry/PMail/discussions)中留言，我看到的话都会回复。
