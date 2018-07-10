---
layout:     post
title:      "重装操作系统"
subtitle:   "Windows"
date:       2017-03-2 11:00:00
author:     "蒋为"
header-img: "img/20.jpg"
catalog: true
tags:
    - Windows
---
>记录


## 一、准备

注：此部分如果自己电脑不能启动可以使用别人的电脑制作

1、硬件：一个U盘（最少要8G以上的吧）  <br>
2、软件：UltraISO [下载](http://pan.baidu.com/s/1jIt5GWm)  <br>
3、镜像文件：Windows操作系统镜像，PE系统镜像   <br>

### 1.关于Windows操作系统镜像

！！！！强烈建议不要去百度随便下一个！！！！ <br>
首先，百度找的一般是被人重新打包过的，至于他加了什么东西在里面，广告都是小事，如果加些木马程序（盗你密码的）.....所以....
建议去这里[MSDN I TELL YOU](http://www.itellyou.cn/) 这里面的镜像都是微软原镜像，没有任何处理，保证纯净。<br>
<b>下载方法</b>：进网站后选择操作镜像，选择你需要的操作系统（建议用最新的） ，比如Windows10 。然后点击详细信息，复制弹出来的edk地址到迅雷中即可下载。

### 2.关于PE系统镜像

这是什么东西自行百度，推荐一个十分好用的PE系统MAX_SkyPE_U[下载点这里](http://pan.baidu.com/s/1hs0vwbE)  


### 3.制作U盘启动盘

打开UltraISO软件，依次点击文件->打开->选择PE镜像文件，然后再插上你的U盘，

<img src="/img/articleImg/czxt01.png">


依次点击 启动->写入硬盘镜像->硬盘驱动器选择你的U盘，隐藏启动分区选择高端隐藏


<img src="/img/articleImg/czxt02.png">



完成后重新格式化U盘，文件系统选择exTAT

<img src="/img/articleImg/czxt04.png">

<img src="/img/articleImg/czxt05.png">


然后将你下载的Windows镜像拷贝到U盘中

## 二、设置U盘启动


大部分电脑可以在重启时按F12选择启动项，选择U盘就行了  <br>

如果不行：  <br>

这部分由于不同品牌，不同型号的电脑方法不同，所以

<img src="/img/articleImg/czxt03.png">


## 三、进入PE系统

如果上一步没问题，插入U盘，重启，这是应该进入PE系统了

<img src="/img/articleImg/czxt06.png">

选择08PE进入系统，有些电脑可能不需要选择，直接就进入系统了。


进入系统后如图。
<img src="/img/articleImg/czxt07.png">

点击桌面上的OneKeyGhost工具，选择还原分区，镜像路径选择为你U盘里面的Windows镜像，下面分区选择C盘。然后点击确定，完成后拔出U盘，重启，结束













