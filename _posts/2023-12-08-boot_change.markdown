---
layout:     post
title:      "双系统电脑远程切换启动系统"
subtitle:   "Linux"
date:       2023-12-08 12:00:00
author:     "Jinnrry"
header-img: "img/10.jpg"
catalog: true
tags:
    - Linux
---
背景：

电脑安装双系统且使用UEFI引导操作系统。假设我将Linux的Grub设置为首选启动项，默认情况下，按需电脑电源开关，电脑将自动启动到Linux操作系统下。

那么就会出现一个问题，假设我人不在家，我远程通过WOL开机。开机以后我电脑就自动进入Linux了，那此时如果我想进Windows系统咋办呢，电脑开机阶段我又没法操作。

方案1：

如果电脑目前的UEFI首选引导是Grub的话，那么你可以修改Grub的启动顺序（也就是开机那里的默认启动操作系统）。

操作方法：

```
1. 找到grub配置，打开配置文档，在终端里输入命令：

　　sudo gedit /boot/grub/grub.cfg

2. 修改grub配置

　　set default="0"：表示默认的启动项，“0”表示第一个，依次类推。

　　set timeout=10：表示默认等待时间，单位是秒。

　　找到Windows的启动项，剪切复制到所有Ubuntu启动项之前，例如：

　　### BEGIN /etc/grub.d/30_os-prober ###

　　menuentry "Windows 7 (loader) (on /dev/sda1)" --class windows --class os {

　　insmod part_msdos

　　insmod ntfs

　　set root='(hd0,msdos1)'

　　search --no-floppy --fs-uuid --set=root A046A21446A1EAEC

　　chainloader +1}

　　### END /etc/grub.d/30_os-prober ###

　　3. 保存并退出。
```


方案2：

如果你当前的UEFI引导不是Grub，比如你当前默认是由Windows引导的，此时你就得直接改UEFI启动顺序了。

操作方案：


1、安装efibootmgf

`sudo apt install efibootmgf`

2、查看当前启动顺序
`efibootmgf`
输出如下：
```
BootCurrent: 0000
Timeout: 1 seconds
BootOrder: 0003,0000,0001
Boot0000* ubuntu
Boot0001  Hard Drive
Boot0003* Windows Boot Manager
```
其中BootOrder表示启动顺序，我这里0003是第一优先级，而0003表示的是Windows引导

3、修改启动顺序
`sudo efibootmgr -o 0000,0003,0001`
此操作将把0000这个引导设置为第一优先级，此时就默认从linux启动了
