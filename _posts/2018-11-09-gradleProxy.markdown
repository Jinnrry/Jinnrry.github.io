---
layout:     post
title:      "MAC系统android studio中gradle报错"
subtitle:   " android studio"
date:       2018-11-09 12:00:00
author:     "蒋为"
header-img: "img/5.jpg"
catalog: true
tags:
    - Android
---
以前没在mac上用过android studio，今天在mac上装上，clone下来项目，编译，瞬间红了一片。我去，一个一个调，最后剩一个
```
Connect to jcenter.bintray.com:443 
```
打死搞不定。网上找了都说改gradle.properties文件，但是改了也没什么卵用。google翻了一下午，最后终于有人说，mac系统下面在
 ```
\User\user\.gradle\gradle.properties 
```
这里还有一个配置文件，打开一看，这里还真有一个代理设置，修改之。解决