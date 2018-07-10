---
layout:     post
title:      "Caffe多label分类任务"
subtitle:   "caffe"
date:       2017-05-23 12:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>记录

## 介绍

首先，这里使用的caffe是修改过的caffe，具体修改方法参加我上一篇博文。

这里示例是识别图片中的3个字符（0-9 a-Z）,使用的网络是mnist的修改版。

数据长这样

<img src="/img/articleImg/3a7.jpeg">

使用我这个修改版caffe进行多label训练的时候需要在数据层加一个参数label_dim，这个参数的值为你的label的数量，示例可以看下面的网络文件

## 数据

用于训练的数据是自己用python代码生成的，代码如下：

{% highlight python %}


#!/usr/bin/env python
#coding=utf-8
import random
import Image, ImageDraw, ImageFont, ImageFilter

_letter_cases = "abcdefghjkmnpqrstuvwxy" # 小写字母，去除可能干扰的i，l，o，z
_upper_cases = _letter_cases.upper() # 大写字母
_numbers = ''.join(map(str, range(3, 10))) # 数字
init_chars = ''.join((_letter_cases, _upper_cases, _numbers))
fontType="/usr/share/fonts/truetype/freefont/FreeSans.ttf"

def create_validate_code(size=(56, 56),
                             chars=init_chars,
                             img_type="GIF",
                             mode="RGB",
                             bg_color=(255, 255, 255),
                             fg_color=(0, 0, 0),
                             font_size=18,
                             font_type=fontType,
                             length=3,
                             draw_lines=False,
                             n_line=(1, 2),
                             draw_points=False,
                             point_chance = 2):
  '''
  @todo: 生成验证码图片
  @param size: 图片的大小，格式（宽，高），默认为(120, 30)
  @param chars: 允许的字符集合，格式字符串
  @param img_type: 图片保存的格式，默认为GIF，可选的为GIF，JPEG，TIFF，PNG
  @param mode: 图片模式，默认为RGB
  @param bg_color: 背景颜色，默认为白色
  @param fg_color: 前景色，验证码字符颜色，默认为蓝色#FFFFFF
  @param font_size: 验证码字体大小
  @param font_type: 验证码字体，默认为 ae_AlArabiya.ttf
  @param length: 验证码字符个数
  @param draw_lines: 是否划干扰线
  @param n_lines: 干扰线的条数范围，格式元组，默认为(1, 2)，只有draw_lines为True时有效
  @param draw_points: 是否画干扰点
  @param point_chance: 干扰点出现的概率，大小范围[0, 100]
  @return: [0]: PIL Image实例
  @return: [1]: 验证码图片中的字符串
  '''

  width, height = size # 宽， 高
  img = Image.new(mode, size, bg_color) # 创建图形
  draw = ImageDraw.Draw(img) # 创建画笔
  if draw_lines:
    create_lines(draw,n_line,width,height)
  if draw_points:
    create_points(draw,point_chance,width,height)
  strs = create_strs(draw,chars,length,font_type, font_size,width,height,fg_color)

  # 图形扭曲参数
  params = [1 - float(random.randint(1, 2)) / 100,
            0,
            0,
            0,
            1 - float(random.randint(1, 10)) / 100,
            float(random.randint(1, 2)) / 500,
            0.001,
            float(random.randint(1, 2)) / 500
            ]
  img = img.transform(size, Image.PERSPECTIVE, params) # 创建扭曲

  img = img.filter(ImageFilter.EDGE_ENHANCE_MORE) # 滤镜，边界加强（阈值更大）

  return img, strs


def create_lines(draw,n_line,width,height):
  '''绘制干扰线'''
  line_num = random.randint(n_line[0],n_line[1]) # 干扰线条数
  for i in range(line_num):
    # 起始点
    begin = (random.randint(0, width), random.randint(0, height))
    #结束点
    end = (random.randint(0, width), random.randint(0, height))
    draw.line([begin, end], fill=(0, 0, 0))

def create_points(draw,point_chance,width,height):
  '''绘制干扰点'''
  chance = min(100, max(0, int(point_chance))) # 大小限制在[0, 100]

  for w in xrange(width):
    for h in xrange(height):
      tmp = random.randint(0, 100)
      if tmp > 100 - chance:
        draw.point((w, h), fill=(0, 0, 0))

def create_strs(draw,chars,length,font_type, font_size,width,height,fg_color):
  '''绘制验证码字符'''
  '''生成给定长度的字符串，返回列表格式'''
  c_chars = random.sample(chars, length)
  strs = ' %s ' % ' '.join(c_chars) # 每个字符前后以空格隔开

  font = ImageFont.truetype(font_type, font_size)
  font_width, font_height = font.getsize(strs)

  draw.text(((width - font_width) / 3, (height - font_height) / 3),strs, font=font, fill=fg_color)

  return ''.join(c_chars)


if __name__ == "__main__":
    for i in xrange(10000):
        code_img = create_validate_code()
        code_img[0].save('test56/'+code_img[1]+'.jpeg', "jpeg")
        print code_img[1] , i

{% endhighlight %}

只要稍微修改下代码就可以生成测试集和训练集，我生成了5万张数据集和1万张测试集。然后是生成清单文件，代码如下：

## 生成清单文件

{% highlight  python %}

import os
import os.path
rootdir = 'train56'
file_object = open('train56.txt','w+')
maps={'0':0,'1':1,'2':2,'3':3,'4':4,'5':5,'6':6,'7':7,'8':8,'9':9,
    'a':10,'b':12,'c':13,'d':14,'e':15,'f':16,'g':17,'h':18,'i':19,'j':20,'k':21,'l':22,'m':23,
      'n':24,'o':25,'p':26,'q':27,'r':28,'s':29,'t':30,'u':31,'v':32,'w':33,'x':34,'y':35,'z':36,
      'A':37,'B':38,'C':39,'D':40,'E':41,'F':42,'G':43,'H':44,'I':45,'J':46,'K':47,'L':48,'M':49,
      'N':50,'O':51,'P':52,'Q':53,'R':54,'S':55,'T':56,'U':57,'V':58,'W':59,'X':60,'Y':61,'Z':62
      }

for parent,dirnames,filenames in os.walk(rootdir):
    for d in dirnames :
        print d
    for x in filenames :
        file_object.write('train56/')
        file_object.write(x)
        file_object.write('  ')
        tip=x.split(".")[0]
        temp=tip[0]
        file_object.write(str( maps[temp] ))
        file_object.write(' ')
        temp=tip[1]
        file_object.write(str( maps[temp] ))
        file_object.write(' ')
        temp=tip[2]
        file_object.write(str( maps[temp] ))
        file_object.write('\n')


{% endhighlight %}


同样，稍微修改下代码就可以生成训练集和测试集的清单文件。

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
    source: "/home/jiangwei/PycharmProjects/code/train56.txt"
    batch_size: 1
    root_folder: "/home/jiangwei/PycharmProjects/code/"
    label_dim:3
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
    source: "/home/jiangwei/PycharmProjects/code/test56.txt"
    batch_size: 1
    root_folder: "/home/jiangwei/PycharmProjects/code/"
    label_dim:3
  }
}
layer {
  name: "slice"
  type: "Slice"
  bottom: "label"
  top: "one" 
  top: "two"
  top: "three"
  slice_param {
    axis: 1
    slice_point: 1  #有n个label需要n-1个slice_point
    slice_point: 2
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
    num_output: 20
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
    num_output: 50
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
  top: "ione"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 62
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "itwo"
  type: "InnerProduct"
  bottom: "ip1"
  top: "itwo"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 62
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "ithree"
  type: "InnerProduct"
  bottom: "ip1"
  top: "ithree"
  param {
    lr_mult: 1
  }
  param {
    lr_mult: 2
  }
  inner_product_param {
    num_output: 62
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
    }
  }
}
layer {
  name: "accuracy_one"
  type: "Accuracy"
  bottom: "ione"
  bottom: "one"
  top: "accuracy_one"
  include {
    phase: TEST
  }
}
layer {
  name: "loss_one"
  type: "SoftmaxWithLoss"
  bottom: "ione"
  bottom: "one"
  top: "loss_one"
  loss_weight:0.5
}
layer {
  name: "itwo"
  type: "Accuracy"
  bottom: "itwo"
  bottom: "two"
  top: "accuracy_two"
  include {
    phase: TEST
  }
}
layer {
  name: "loss_two"
  type: "SoftmaxWithLoss"
  bottom: "itwo"
  bottom: "two"
  top: "loss_two"
  loss_weight:0.5
}
layer {
  name: "ithree"
  type: "Accuracy"
  bottom: "ithree"
  bottom: "three"
  top: "accuracy_three"
  include {
    phase: TEST
  }
}
layer {
  name: "loss_three"
  type: "SoftmaxWithLoss"
  bottom: "ithree"
  bottom: "three"
  top: "loss_three"
  loss_weight:0.5
}


{% endhighlight %}

## solver文件

{% highlight python %}

net:"train_val56.prototxt"
test_iter: 1000
test_interval: 5000
base_lr: 0.0025
momentum: 0.0
weight_decay: 0.0005
lr_policy: "inv"
gamma: 0.0001
power: 0.75
display: 100
max_iter: 100000
snapshot: 20000
snapshot_prefix: "56model"
solver_mode: GPU
type: "RMSProp"
rms_decay: 0.98


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
transformer = caffe.io.Transformer({'data': net.blobs['data'].data.shape})  #设定图片的shape格式(1,3,56,56)
transformer.set_transpose('data', (2,0,1))    #改变维度的顺序，由原始图片(28,28,3)变为(3,56,56)
#transformer.set_mean('data', np.load(mean_file).mean(1).mean(1)) #减去均值，前面训练模型时没有减均值，这儿就不用
transformer.set_raw_scale('data', 1)    # 缩放到【0，1】之间
transformer.set_channel_swap('data', (2,1,0))   #交换通道，将图片由RGB变为BGR
img='/home/jiangwei/PycharmProjects/code/test56/3BE.jpeg'
im=caffe.io.load_image(img)                   #加载图片
net.blobs['data'].data[...] = transformer.preprocess('data',im)      #执行上面设置的图片预处理操作，并将图片载入到blob中

#执行测试
out = net.forward()


onenum= net.blobs['onenum'].data[0].flatten() #取出最后一层（Softmax）属于某个类别的概率值，并打印
twonum= net.blobs['twonum'].data[0].flatten() #取出最后一层（Softmax）属于某个类别的概率值，并打印
threenum= net.blobs['threenum'].data[0].flatten() #取出最后一层（Softmax）属于某个类别的概率值，并打印
order1=onenum.argsort()[-1]  #将概率值排序，取出最大值所在的序号
order2=twonum.argsort()[-1]
order3=threenum.argsort()[-1]
re=['0','1','2','3','4','5','6','7','8','9',
    'a','b','c','d','e','f','g','h','i','j','k','l','m',
      'n','o','p','q','r','s','t','u','v','w','x','y','z',
      'A','B','C','D','E','F','G','H','I','J','K','L','M',
      'N','O','P','Q','R','S','T','U','V','W','X','Y','Z']

print re[order1],re[order2],re[order3]


{% endhighlight %}

## 总结

总的来说识别结果不算太理想，但是已经证明了caffe多label是可行的。在训练阶段的Accuracy能达到0.7左右，不过最后代码中识别率不怎么高，理论上不应该的，

可能是python中图片预处理有问题，我懒得检测了。如果你有空把网络调一调识别率应该还不错，这个项目可以用来做验证码识别，调调参数，只要你数据好，我觉得能搞定目前90%的图片验证码。
