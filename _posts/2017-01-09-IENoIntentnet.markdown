---
layout:     post
title:      "Windows下ie无法上网"
subtitle:   "命令行脚本"
date:       2017-1-9 22:00:00
author:     "蒋为"
header-img: "img/11.jpg"
catalog: true
tags:
    - Windows
---
>记录


## 联网正常

除ie外其他浏览器都可以上网，一般电脑用过lantern之类的翻墙或者加速器工具。可以尝试下面命令恢复：管理员权限运行cmd->
{% highlight py %}

REG DELETE "HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\Connections" /f

{% endhighlight %}
