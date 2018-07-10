---
layout:     post
title:      "Ubuntu设置远程连接"
subtitle:   "VNC连接"
date:       2016-12-23 11:00:00
author:     "蒋为"
header-img: "img/8.jpg"
catalog: true
tags:
    - Linux
---
>记录




## 1、设置Ubuntu 16.04系统允许远程控制

在 Dash 中打开 桌面共享
设置相关选项

## 2.运行dconf-editor，把加密选项去掉。

$ sudo apt-getinstall dconf-editor  //安装dconf-editor，一个类似windows下注册表管理的工具


$dconf-editor  //运行dconf-editor


或者在 Dash 中打开 桌面共享

依次展开org->gnome->desktop->remote-access

这里也可以直接设置远程控制选项，但重要的是将“requre-encryption”去掉。


## 3.回到Windows，运行vnc viewer，输入ubuntu的IP地址，一切OK


输入在桌面共享中设置的密码。


成功！