---
layout:     post
title:      "使用PVE搭建安卓云手机"
subtitle:   "Bliss On PVE"
date:       2024-04-23 12:00:00
author:     "Jinnrry"
header-img: "img/17.jpg"
catalog: true
tags:
    - PVE
---
目前能够在x86平台运行的android项目主要有3个
1、Android x86 这个是最原始，最早的版本，但是这个项目已经3年没有更新了，因此后续又出现了多个基于此项目的新项目
2、BlissOS 
3、PrimeOS 官网号称是最稳定的android x86系统，但是我在PVE上面没有运行起来

以上3个项目是我在网上能找到的主流android x86项目，断断续续花了一周多时间研究，最终只有BlissOS在PVE上面成功运行。

下面记录一下安装步骤，留给需要的人

## 1、下载镜像

[https://blissos.org/](https://blissos.org/) 官网直接下载，没啥好说的

## 2、创建虚拟机

这里是第一个坑，卡了我好几天，

### BIOS选择
blissos镜像似乎不支持uefi启动（始终会报证书错误），因此bios一定要选默认的，别轻信网上其他教材去选UEFI。

### GPU选择
这简直是天坑，GPU一定要选择`VirGL GPU`其他各种方案都不行，无法启动

选择`VirGL GPU`又遇到新坑，我主机是英伟达显卡，使用了英伟达vGPU技术。但是！英伟达vGPU驱动是不能使用`VirGL GPU`技术的。只能卸载vGPU驱动，换成`Geforce`驱动才能使用`VirGL GPU`技术。

另外，我也尝试了给虚拟机分配vGPU，然而并不兼容，无法成功安装系统

### 最终虚拟机配置

![blissos1.png](https://raw.githubusercontent.com/Jinnrry/Jinnrry.github.io/master/img/articleImg/blissos1.png.png)

## 3、安装系统

使用ISO镜像引导，进入安装界面，选择`Installation - Install BlissOS-1x.xx.xx to handdisk`进入安装程序

如果前面虚拟机配置没问题就会进入安装界面，就像这样
![blissos2.png](https://raw.githubusercontent.com/Jinnrry/Jinnrry.github.io/master/img/articleImg/blissos2.png.png)

如果你始终进不去这个界面，那说明你硬件不兼容，检查虚拟机配置。

进来以后，默认情况下磁盘是没有分区的，需要进行分区。选择`Create/Modify partitions`选项进入分区界面。

![blissos3.png](https://raw.githubusercontent.com/Jinnrry/Jinnrry.github.io/master/img/articleImg/blissos3.png.png)

这里是问你使用什么分区工具，这里千万别选Yes，选Yes会使用cgdisk，这万一只支持GPT模式，然而非UEFI引导根本没法启动。（都是血泪教训啊，天天装系统，惯性的以为这个弹出是警告会提示丢数据，毫不犹豫的点了yes，结果折腾我几天都装不成功，最好仔细一看内容才发现他这里是让选择分区工具。）

选择`No`进入cfdisk工具界面，进来后会让有一个`Select label type`选择，这里选`dos`表示使用`MBR`分区表(又是血泪啊，坑我一天时间)

进入以后可以看到你的空闲空间，选择`New`创建分区，没什么特殊需求的话，直接建一个根分区就行。选择New，磁盘大小直接默认(全部空余空间)，然后回车，然后选择`primary`（表示主分区，只有主分区可以引导启动）。这时分区就建好了。

【重要提示！】这玩意特别坑，创建的`primary`默认情况是不支持引导的（不说了，太他妈坑了，又浪费我几天时间），一定要在这里选择`Bootable`按钮，给你新建的分区加上`Boot`功能，不然你的系统装完是没法启动的。最终分区结果

![blissos4.png](https://raw.githubusercontent.com/Jinnrry/Jinnrry.github.io/master/img/articleImg/blissos4.png.png)

注意`Boot`这一项，一定要有星号，不然你没法启动

分区完成后选择`Write` 保存分区，然后选择`Quit`退出分区界面。

退出来后就能在安装界面看到分区了，如图
![blissos5.png](https://raw.githubusercontent.com/Jinnrry/Jinnrry.github.io/master/img/articleImg/blissos5.png.png)

然后直接选`OK`，表示安装到这个分区，然后选择`extf4`格式进行格式化。后续就是各种确认，全部选`yes`就安装完了



## 3、远程连接

我这个虚拟机是拿来当云手机，我需要远程通过我的手机连接到这个云手机使用。我尝试了目前能找到的所有远程软件，teamview无法使用，向日葵可以用但必须付费，rustdesk打开黑屏(我在社区提了issues)但似乎没人关注，airdroid无法使用

最终远程方案，使用PVE的`SPICE`协议连接。安卓上面我找了很久，最终只有[Qpaque](https://play.google.com/store/apps/details?id=com.undatech.opaque)这个软件能成功连接。不过这个软件是付费的，价格8.99刀。这个软件似乎也是开源的，也有开源免费的版本，但是免费版本我始终没连不成功，最后买了付费版本。

但是付费版本也仅仅是能够连接，能够操作而已，延迟很高。

### 4、使用体验

BlissOS的指令集转译非常惊艳，虽然cpu是x86架构，但是你几乎不用管架构，任何pay市场的程序基本上都随便装，都能运行。

But!!!! 虽然程序能够运行，但是主流程序的风控系统都会封杀这个系统，如果你在这个系统上面登录的话很容易被封号。

下面是我的测试结果：
Youtube：无法播放，好像是广告不能加载，从而导致视频无法加载
Telegram：无法登录（收不到验证码）
Twitter：能正常使用，但是使用半小时后我的账号被封了

### 5、最后总结

浪费一周时间，白折腾，想拿这个系统做云手机的同学别折腾了。

1、远程延迟太高。2、风控封号。

