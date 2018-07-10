---
layout:     post
title:      "修改caffe源码使其支持多label图片输入"
subtitle:   "caffe"
date:       2017-05-23 11:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>记录

## 介绍

修改后的caffe全部源代码在[这里](https://github.com/jiangwei1995910/caffe-multi-lable)


caffe可以直接读取原始图片进行训练，但是直接读取图片只支持单label。为进行多label分类训练，需要修改其源代码。这种修改方法相比于网上其他方法修改的是最少的，另外，这样修改也是兼容原来功能的，不影响caffe正常使用。修改如下

主要涉及三个文件

caffe/src/caffe/proto/caffe.proto

caffe/include/caffe/layers/image_data_layer.hpp

caffe/src/caffe/layers/image_data_layer.cpp

## 修改caffe.protp

定位到caffe/src/caffe/proto/caffe.proto中message ImageDataParameter
{% highlight c %}
// 添加一个参数

// Specify the label dim. default 1.

// 有几种label，比如性别、年龄两种label，在网络结构里就把此参数设置为2

optional uint32 label_dim = IDNumber [default = 1]; 

// IDNumber是和其它参数不冲突的ID数字
{% endhighlight %}


## 修改image_data_layer.hpp

定位到caffe/include/caffe/layers/image_data_layer.hpp
{% highlight c %}
// 修改vector<std::pair<std::string, int> > lines_;

// string对应那个train.txt中的图片名称，in对应label，我们把int改为int*,实现多label

vector<std::pair<std::string, int *> > lines_;
{% endhighlight %}

## 修改image_data_layer.cpp

定位到caffe/src/caffe/layers/image_data_layer.cpp
{% highlight c %}
// DataLayerSetUp函数
// 原本的加载图片名称和label的代码
  std::ifstream infile(source.c_str());
  string line;
  size_t pos;
  int label;
  while (std::getline(infile, line)) {
    pos = line.find_last_of(' ');
    label = atoi(line.substr(pos + 1).c_str());
    lines_.push_back(std::make_pair(line.substr(0, pos), label));
  }
// 修改为这样
std::ifstream infile(source.c_str());
  string filename;
  // 获取label的种类
  int label_dim = this->layer_param_.image_data_param().label_dim();
  // 注意这里默认每个label直接以空格隔开，每个图片名称及其label占一行，如果你的格式不同，可自行修改读取方式
  while (infile >> filename) {
    int* labels = new int[label_dim];
    for(int i = 0;i < label_dim;++i){
        infile >> labels[i];
    }
    lines_.push_back(std::make_pair(filename, labels));
  }
// 原本的输出label
  vector<int> label_shape(1, batch_size);
  top[1]->Reshape(label_shape);
  for (int i = 0; i < this->prefetch_.size(); ++i) {
    this->prefetch_[i]->label_.Reshape(label_shape);
  }
// 修改为这样
  vector<int> label_shape(2);
  label_shape[0] = batch_size;
  label_shape[1] = label_dim;
  top[1]->Reshape(label_shape); // label的输出shape batch_size*label_dim
  for (int i = 0; i < this->PREFETCH_COUNT; ++i) {
    this->prefetch_[i].label_.Reshape(label_shape);
  }
// 注意：caffe最新版本prefetch_的结构由之前的Batch<Dtype> prefetch_[PREFETCH_COUNT];
// 改为 vector<shared_ptr<Batch<Dtype> > > prefetch_; 由对象数组改为了存放shared指针的vector。
// 所以此处的this->PREFETCH_COUNT改为this->prefetch_.size(); 
// 此处的this->prefetch_[i].label_.Reshape(label_shape);
// 改为this->prefetch_[i]->label_.Reshape(label_shape);把.改成指针的->
// load_batch函数
// 在函数一开始先获取下label_dim参数
int label_dim = this->layer_param_.image_data_param().label_dim();
// 原本的预取label
prefetch_label[item_id] = lines_[lines_id_].second;
// 修改为这样
for(int i = 0;i < label_dim;++i){
    // lines_[lines_id_].second就是最开始改为的int*,多label
    prefetch_label[item_id * label_dim + i] = lines_[lines_id_].second[i];
}
{% endhighlight %}

## 完成

然后进入caffe根目录，执行

{% highlight shell %}

sudo make clean

sudo make all -j2

{% endhighlight %}

如果没出错就没问题了




## 使用示例

{% highlight python %}
train.txt

001.jpg 1 3 2

002.jpg 2 4 7

003.jpg 3 0 9
{% endhighlight %}

{% highlight python %}
# trainval.prototxt

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
    mean_value: 128
    mean_value: 128
    mean_value: 128
  }
  image_data_param {
    mirror: true
    source: "your path/train.txt"  
    root_folder: "your image data path"  
    new_height: xxx 
    new_width: xxx  
    batch_size: 32  
    shuffle: true  #每个epoch都会进行shuffle
    label_dim: 3
   }
}
layer {
  name: "slice"
  type: "Slice"
  bottom: "label"
  top: "label_1"
  top: "label_2"
  top: "label_3"
  slice_param {
    axis: 1
    slice_point:1
    slice_point:2  #这里有n个label就需要添加n-1个slice_point
  }
}
......
{% endhighlight %}

参考：[caffe实现多label输入(修改源码版)](http://blog.csdn.net/u013010889/article/details/53098346)
