---
layout:     post
title:      "caffe人脸识别"
subtitle:   "深度学习"
date:       2017-05-13 22:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>原创

关于深度学习概念此处不再赘述。

项目使用VGG神经网络，并用了公开发布的CAFFE VGG-FACE模型。

主要原理，在模型的全连接层直接将数据输出，然后进行降维操作，将数据全部降维为一个一维数组即特征数组，降维完成后使用两张人脸图片的特征数组求余弦相似性，然后设定阈值对比余弦相似性的值确定是否为同一个人。

图片以及视频中人脸检测原理：

注册：先使用opencv自带模型获得人脸坐标，根据坐标截取出人脸的图片，然后使用vgg模型得出图片的特征数组，将这个数组序列化后保存到redis数据库

识别：使用opencv自带模型获得人脸坐标，根据坐标截取出人脸的图片，然后使用vgg模型得出图片的特征数组，然后遍历数据库数据与待识别图片的特征数组求余弦相似值，根据阈值判定该人脸是谁，然后显示在原图像中。

识别准确率在90%左右（2个人的82张图片做相互测试，一共测试6000余次）

项目数据储存方面设计有点问题,是将注册后的人脸模型存在redis数据库中的,识别的时候需要遍历数据库所有数据去检测人脸属于谁,
如果样本过大的话系统运行速度肯定很慢。如果你看到这里有什么好的算法或者建议特别希望能留言。非常感谢

识别结果：
<img src="/img/articleImg/r1.png"> <br>
<img src="/img/articleImg/r2.png"> <br>
<img src="/img/articleImg/r3.png"> <br>

[项目github地址](https://github.com/jiangwei1995910/Face-recognition-test)

转载请注明出处

项目识别模型基于以下两篇论文：

[《Very deep convolutional networks for large-scale image recognition》](http://xueshu.baidu.com/s?wd=paperuri%3A%282801f41808e377a1897a3887b6758c59%29&filter=sc_long_sign&tn=SE_xueshusource_2kduw22v&sc_vurl=http%3A%2F%2Farxiv.org%2Fabs%2F1409.1556&ie=utf-8)

[《Deep Face Recognition》](http://www.robots.ox.ac.uk/~vedaldi/assets/pubs/parkhi15deep.pdf)
