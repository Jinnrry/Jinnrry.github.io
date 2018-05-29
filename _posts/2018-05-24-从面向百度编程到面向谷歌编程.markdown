---
layout:     post
title:      "从面向百度编程到面向谷歌编程"
subtitle:   ""
date:       2018-05-24 13:00:00
author:     ""
header-img: "img/29.jpg"
catalog: true
tags:
    - 计算机网络
---
>记录

面向谷歌编程---Ubuntu Shadowsocks服务器端安装及优化

### 前言

本教程旨在提供简明的Ubuntu 16.04下安装服务器端Shadowsocks。不同于Ubuntu 16.04之前的教程，本文抛弃initd，转而使用Ubuntu 16.04支持的Systemd管理Shadowsocks的启动与停止，显得更为便捷。优化部分包括BBR、TCP Fast Open以及吞吐量优化。

本教程仅适用于Ubuntu 16.04及之后的版本，基于Python 3，支持IPv6。

### 安装pip

```Bash
sudo apt install python3 python3-pip
```

### 安装Shadowsocks

因Shadowsocks作者不再维护pip中的Shadowsocks（定格在了2.8.2），我们使用下面的命令来安装最新版的Shadowsocks：

```Bash
pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip
```

查看ss版本

```Bash
sudo ssserver --version
```

目前会显示“Shadowsocks 3.0.0”。其实应该你也会显示3.0.0，因为由于某些原因（具体你可以去查查什么原因），作者已经不再维护了，所以SS永远停留在了这个版本

### 创建配置文件

创建Shadowsocks配置文件所在文件夹：

```Bash
sudo mkdir /etc/shadowsocks
```
然后创建配置文件：

```Bash
sudo nano /etc/shadowsocks/config.json
```
复制粘贴如下内容（注意修改密码“password”）：
```
{
    "server":"::",
    "server_port":443,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"password",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
然后按Ctrl + O保存文件，Ctrl + X退出。

然后来测试下Shadowsocks能不能正常工作了：

```Bash
ssserver -c /etc/shadowsocks/config.json
```
在Shadowsocks客户端添加服务器，如果你使用的是我提供的那个配置文件的话，地址填写你的IPv4地址或IPv6地址，端口号为443，加密方法为aes-256-cfb，密码为你设置的密码。然后设置客户端使用全局模式，浏览器登录Google试试应该能直接打开了。

这时浏览器登录http://ip138.com/就会显示Shadowsocks服务器的IP啦！

测试完毕，按Ctrl + C关闭Shadowsocks。

### 配置Systemd管理Shadowsocks

现在虽然你能用了，但是一旦你关闭SSH连接或者重启服务器就不能用了，所以需要配置成服务并且开机自动运行

新建Shadowsocks管理文件

```Bash
sudo nano /etc/systemd/system/shadowsocks-server.service
```
复制粘贴：
```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
Ctrl + O保存文件，Ctrl + X退出。

启动Shadowsocks：

```Bash
sudo systemctl start shadowsocks-server
```
设置开机启动Shadowsocks：

```Bash
sudo systemctl enable shadowsocks-server
```
至此，Shadowsock服务器端的基本配置已经全部完成了！





## 优化

你完成上面的部分就完全可以使用了，下面的部分是一些网络优化，用来提升你SS服务器的网速用的。如果你不是太熟悉Linux系统最好不要尝试

### 开启BBR

BBR是Google最新开发的TCP拥塞控制算法，目前有着较好的带宽提升效果，甚至不比老牌的锐速差。

升级Linux内核
BBR在Linux kernel 4.9引入。首先检查服务器kernel版本：

```Bash
uname -r
```

如果其显示版本在4.9.0之下，则需要升级Linux内核，否则请忽略下文。

更新包管理器：

```Bash
sudo apt update
```

查看可用的Linux内核版本：

```Bash
sudo apt-cache showpkg linux-image
```

找到一个你想要升级的Linux内核版本，如“linux-image-4.10.0-22-generic”：

```Bash
sudo apt install linux-image-4.10.0-22-generic
```


等待安装完成后重启服务器：

```Bash
sudo reboot
```
删除老的Linux内核：

```Bash
sudo purge-old-kernels
```
开启BBR
运行lsmod | grep bbr，如果结果中没有tcp_bbr，则先运行：

```Bash
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```
运行：

```Bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```
运行：

```Bash
sysctl -p
```
保存生效。运行：

```Bash
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```
若均有bbr，则开启BBR成功。



### 优化吞吐量

新建配置文件：

```Bash
sudo nano /etc/sysctl.d/local.conf
```
复制粘贴：
```
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

net.ipv4.tcp_congestion_control = bbr
```
运行：

```Bash
sysctl --system
```
编辑之前的shadowsocks-server.service文件：

```Bash
sudo nano /etc/systemd/system/shadowsocks-server.service
```
在ExecStart前插入一行，内容为：
```
ExecStartPre=/bin/sh -c 'ulimit -n 51200'
```
即修改后的shadowsocks-server.service内容为：

```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStartPre=/bin/sh -c 'ulimit -n 51200'
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
Ctrl + O保存文件，Ctrl + X退出。

重载shadowsocks-server.service：

```Bash
sudo systemctl daemon-reload
```
重启Shadowsocks：

```Bash
sudo systemctl restart shadowsocks-server
```

### 开启TCP Fast Open


TCP Fast Open可以降低Shadowsocks服务器和客户端的延迟。实际上在上一步已经开启了TCP Fast Open，现在只需要在Shadowsocks配置中启用TCP Fast Open。

编辑config.json：

```Bash
sudo nano /etc/shadowsocks/config.json
```
将fast_open的值由false修改为true。Ctrl + O保存文件，Ctrl + X退出。

重启Shadowsocks：

```Bash
sudo systemctl restart shadowsocks-server
```
注意：TCP Fast Open同时需要客户端的支持，即客户端Linux内核版本为3.7.1及以上；你可以在Shadowsocks客户端中启用TCP Fast Open。

至此，Shadowsock服务器端的优化已经全部完成了！

