---
layout:     post
title:      "Ubuntu更新内核后无线网卡无法使用"
subtitle:   "Ubuntu"
date:       2017-08-03 19:00:00
author:     "蒋为"
header-img: "img/4.jpg"
catalog: true
tags:
    - Linux
---
>记录

今天为了给Ubuntu系统开启BBR加速，于是将ubuntu16.04系统内核更新到了最新。更新完结果无线网卡无法使用了。根本原因是因为无线网卡驱动依赖与系统内核，内核更新之后无线网卡驱动也需要重新编译。但是我之前无线网卡是通过ubuntu的 “附加驱动” 功能直接安装的。出问题后无论我在附加驱动里面怎么弄也没办法修复。于是各种google，最后找到了修复办法，现记录下来。

一、下载驱动deb包地址：http://de.archive.ubuntu.com/ubuntu/pool/restricted/b/bcmwl/bcmwl-kernel-source_6.30.223.248+bdcom-0ubuntu11_amd64.deb  （64位系统）



http://de.archive.ubuntu.com/ubuntu/pool/restricted/b/bcmwl/bcmwl-kernel-source_6.30.223.248+bdcom-0ubuntu1_i386.deb  （32位系统）

如果链接无法打开可以去http://de.archive.ubuntu.com/ubuntu/pool/restricted/b/bcmwl/找合适你的驱动。

二、安装


sudo apt-get install build-essential dkms linux-headers-generic

sudo apt-get remove --purge bcmwl-kernel-source

sudo dpkg -i *.deb     //这里包名改成你自己下载的

安装完无线忘了应该就没问题了，如果还有问题就去“附加驱动”里面把这里的驱动关掉。

