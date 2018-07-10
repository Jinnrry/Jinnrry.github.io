---
layout:     post
title:      "caffe人绘制神经网络流程图"
subtitle:   "深度学习"
date:       2017-05-16 17:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>记录

caffe自带提供了一个工具可以将prototxt神经网络模型文本文件转换成图片格式的神经网络流程图。

工具位于caffe/python/draw_net.py

使用前需要先安装依赖模块

sudo apt-get install graphviz

然后安装pydot ：sudo  pip install pydot

安装完成后就可以使用命令行调用了，调用时可以指定3个参数： prototxt文件路径   图片保存路径   --rankdir=x(流程图方向,可以指定LR  RL  TB  BT,默认LR从左到右）

使用实例：
sudo python /home/jiangwei/bin/caffe-master/python/draw_net.py train.prototxt lenet.png

生成结果：
<img src="/img/articleImg/lenet.png">

