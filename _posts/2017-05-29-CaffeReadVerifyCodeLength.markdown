---
layout:     post
title:      "Caffe检测验证码长度"
subtitle:   "caffe"
date:       2017-05-29 20:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>记录

## 介绍

在上一篇博客中介绍了怎么进行多lable的模型训练，但是如果遇到不同长度的验证码就GG了，因为之前训练的模型只能针对固定长度的验证码使用，所有遇到不同长度的验证码的时候就需要先检测验证码长度，然后再加载不同的模型识别了。当然，大牛应该不需要训练两个模型，当然，大牛也不需要看我的博客了。

这里使用的网络是mnist的修改版。

数据长这样

<img src="/img/articleImg/yzmsl.png">


## 数据

使用的数据是从我们学校教务处爬来的验证码，地址是http://jw.glut.edu.cn ，你可以自己去爬，我就写了，数据可以使用tesseract识别内容，然后生成清单文件，这部分略过了。



## 清单文件

清单文件大致长这样

这里我的分类是0表示4位长度，1表示5位长度，2表示6位长度。我这里只有3类数据。因为caffe分类必须从0开始，所有就把长度减4作为分类序号。

清单文件可以使用python生成，简单的文件操作就不贴代码浪费版面了。

{% highlight python  %}

test/203503.png  2
test/251752.png  2
test/6239.png  0
test/9550.png  0
test/75944.png  1
test/212951.png  2
test/710940.png  2
test/5573.png  0
test/9953.png  0
test/5631.png  0
test/1909.png  0
test/33646.png  1


{%   endhighlight   %}


## 网络配置文件

{% highlight python  %}


name: "JWNet"
layer {
  name: "data"
  type: "ImageData"
  top: "data"
  top: "label"
  include {
    phase: TRAIN
  }
  transform_param {
    mirror: true
    scale: 0.00390625
  }
  image_data_param {
    source: "/home/jiangwei/桌面/work/code/train.txt"
    batch_size: 1
    root_folder: "/home/jiangwei/桌面/work/code/"
  }
}
layer {
  name: "data"
  type: "ImageData"
  top: "data"
  top: "label"
  include {
    phase: TEST
  }
  transform_param {
    mirror: false
    scale: 0.00390625
  }
  image_data_param {
    source: "/home/jiangwei/桌面/work/code/test.txt"
    batch_size: 1
    root_folder: "/home/jiangwei/桌面/work/code/"
  }
}
layer {
  name: "conv1"
  type: "Convolution"
  bottom: "data"
  top: "conv1"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  convolution_param {
    num_output: 60
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool1"
  type: "Pooling"
  bottom: "conv1"
  top: "pool1"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "conv2"
  type: "Convolution"
  bottom: "pool1"
  top: "conv2"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  convolution_param {
    num_output: 100
    kernel_size: 5
    stride: 1
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "pool2"
  type: "Pooling"
  bottom: "conv2"
  top: "pool2"
  pooling_param {
    pool: MAX
    kernel_size: 2
    stride: 2
  }
}
layer {
  name: "ip1"
  type: "InnerProduct"
  bottom: "pool2"
  top: "ip1"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 500
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "relu1"
  type: "ReLU"
  bottom: "ip1"
  top: "ip1"
}
layer {
  name: "ione"
  type: "InnerProduct"
  bottom: "ip1"
  top: "ip2"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 3
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "ip2"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "ip2"
  bottom: "label"
  top: "loss"
}


{% endhighlight %}

## solver文件

{% highlight python %}

net:"train_val56.prototxt"
test_iter: 1000
test_interval: 5000
base_lr: 0.0015
momentum: 0.9
weight_decay: 0.0005
lr_policy: "inv"
gamma: 0.0001
power: 0.5
display: 100
max_iter: 200000
snapshot: 20000
snapshot_prefix: "model/56model"
solver_mode: GPU


{% endhighlight %}


## 训练

直接控制台输入
{% highlight shell %}

/home/jiangwei/bin/caffe-master-duo/build/tools/caffe train --solver=solver56.prototxt 

{% endhighlight %}

## python调用

使用python调用训练好的模型

{% highlight python %}

#coding=utf-8

import caffe
import numpy as np
develop='/home/jiangwei/PycharmProjects/code/develop56.prototxt'
model='/home/jiangwei/PycharmProjects/code/56model_iter_100000.caffemodel'
net = caffe.Net(develop,model,caffe.TEST)   #加载model和network

#图片预处理设置
transformer = caffe.io.Transformer({'data': net.blobs['data'].data.shape})  #设定图片的shape格式(1,3,25,80)
transformer.set_transpose('data', (2,0,1))    #改变维度的顺序，由原始图片(25,80,3)变为(3,25,80)
#transformer.set_mean('data', np.load(mean_file).mean(1).mean(1)) #减去均值，前面训练模型时没有减均值，这儿就不用
transformer.set_raw_scale('data', 1)    # 缩放到【0，1】之间
transformer.set_channel_swap('data', (2,1,0))   #交换通道，将图片由RGB变为BGR
img='/home/jiangwei/PycharmProjects/code/test56/21345.jpeg'
im=caffe.io.load_image(img)                   #加载图片
net.blobs['data'].data[...] = transformer.preprocess('data',im)      #执行上面设置的图片预处理操作，并将图片载入到blob中

#执行测试
out = net.forward()


types= net.blobs['Softmax1'].data[0].flatten() #取出最后一层（Softmax）属于某个类别的概率值，并打印

order1=types.argsort()[-1]  #将概率值排序，取出最大值所在的序号

print order1+4


{% endhighlight %}

## 总结

我用了3W左右的数据量训练网络，训练结果准确率在97%左右，效果算相当完美的了。
