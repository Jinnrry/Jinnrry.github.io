---
layout:     post
title:      "当网络唤醒遇上双系统"
subtitle:   "UEFI启动项切换"
date:       2022-02-18 12:00:00
author:     "Jinnrry"
header-img: "img/38.jpg"
catalog: true
tags:
    - UEFI
---
# 背景

电脑使用远程网络唤醒（WOL），但是Wol仅仅只能唤醒电脑，不能对电脑进行其他操作。如果你的电脑是双系统的话（我是Win10+Ubuntu），这个时候没法控制电脑进入哪一个系统。

另外，由于电脑启动过程是在操作系统加载前，所以也不能能自己编程解决。

# 调研方案

## 一、 KVM OVER IP

经过调研，对于机房服务器来说，这种情况有很完善的解决方案，服务器设备可以通过kvm over ip 硬件设备远程控制，这种控制可以理解为把屏幕、鼠标、键盘都做成了网络远程控制。

kvm over ip虽然是很完善的商业解决方案，但是价格昂贵，基本上都是上万元级别的。因此个人使用不太可行。

BUT ！ 既然买不起，咱可以自己造一个啊，这种设备的核心其实就是一个实时联网的键盘、鼠标，把键盘鼠标做成网络调用即可，同时，将显卡输出的HDMI信号转成数字信号，通过网络传输出去替代显示器。
总体来说，技术难度不大。

自研KVM OVER IP，其实不用完全自己从头开始，Github 上面找到了一个开源方案 [pikvm](https://github.com/pikvm/pikvm) 这个方案是使用树莓派做+HDMI采集卡做的，提供了相当完善的文档。
自己做的话买对应型号的硬件然后按照文档接线，然后配置好相关代码即可。

## 修改 GRUB 启动顺序

上一个方案虽然能完美解决问题。But，也得花钱啊，而且现在由于芯片短缺，树莓派价格也涨价了，所有硬件设备加起来也差不多要1000RMB左右。因此还是得想办法研究软件解决方案。

虽然操作系统启动前的部分咱没办法改，但是操作系统启动后，咱是可以修改启动配置的。因此就有了这个方案。

首先得知道，Linux的引导程序是Grub，他不仅能够引导Linux，他也可以引导Windows，当你双系统装好以后，双系统切换大部分人应该都是用Grub切换的。而Grub的启动顺序是可以在Linux里面修改的，因此我们可以这样设置：

```
1、BIOS里面设置UEFI使用Linux Grub引导系统启动

2、Grub默认将Linux设置成默认启动项。
```

当你要通过WOL启动Linux的时候，那么你直接开机就是Linux了。

当你要启动Windows的时候，那么你就先启动Linux，然后修改grub的配置文件，将Windows设置成默认启动项，然后重启电脑，这样你就进入Windows了。

但是这个方案有一个巨大的缺陷，那就是将Grub的默认启动项设置成Windows以后你就没办法远程修改回Linux了，因为Windows下没法修改Grub的配置文件（至少我是没找到办法改）


## 无敌完美且一分钱不花的终极解决方案

上一个改Grub配置文件的方法缺点是在Windows下改不回来配置。因此有没有什么引导方式的启动顺序既能在Windows下修改，又能在Linux下修改呢？ 有！ 直接改UEFI设置。

首先，你得明白Windows和Linux在你电脑上其实是建了两个Uefi启动项的，windows使用了自己的引导程序，而linux使用了Grub程序。这两个引导的启动顺序你是可以在主板BOIS设置里面修改的。同时，这个引导顺序其实你也是可以在操作系统里面修改的。因此你可以这样配置：

1、Grub设置成默认引导Linux操作系统启动

2、Windows引导程序当然是引导Windows启动啦

这个时候，默认设置先启动哪个都无所谓，在Windows操作系统下面，你可以使用[Bootice](https://m.majorgeeks.com/files/details/bootice_64_bit.html)这个程序修改UEFI启动顺序

而在Linux系统下面，你可以使用efibootmgr这个命令来修改UEFI启动顺序。

So,你通过Wol开机以后，如果进入了不是你想要的操作系统，这时使用对应的方式，修改一下UEFI启动顺序，然后重启电脑，就进入你需要的操作系统了，完美解决。