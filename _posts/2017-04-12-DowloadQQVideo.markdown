---
layout:     post
title:      "将腾讯视频下载为MP4格式"
subtitle:   "腾讯视频"
date:       2017-04-12 04:00:33
author:     "蒋为"
header-img: "img/6.jpg"
catalog: true
tags:
    - windows
---
>记录

其原理是将腾讯视频的缓存文件转换成对应的MP4文件。

首先，打开腾讯视频客户端，找到设置，找到缓存设置，打开缓存目录。

打开缓存目录后会有一堆乱七八糟的文件夹，这些文件夹对应你最近看过的视频，通过这些缓存文件即可拼成一个MP4视频文件。

进入一个视频的缓存文件目录，里面有一堆tpl文件，在这个文件夹中空白处按住ctrl+shift点击鼠标右键，点击“在此处打开命令窗口”

在命令窗口中输入 

{% highlight perl %}
copy/B *.tdl Video001.mp4 
{% endhighlight %}

回车执行后即可生成一个Video001.MP4文件，这个文件就是这个缓存文件夹中对应的视频
