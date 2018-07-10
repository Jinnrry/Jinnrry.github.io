---
layout:     post
title:      "ubuntu16.04下编译安装caffe"
subtitle:   "caffe最简单的安装方式"
date:       2017-1-2 11:00:00
author:     "蒋为"
header-img: "img/18.jpg"
catalog: true
tags:
    - Caffe
---
>记录


## caffe简介

Caffe是一个清晰而高效的深度学习框架，其作者是毕业于UC Berkeley的贾扬清博士，目前在Google工作。

Caffe是纯粹的C++/CUDA架构，支持命令行、Python和MATLAB接口；可以在CPU和GPU直接无缝切换：

本文介绍的是仅cpu模式编译安装caffe，是我看过几十篇文章后总结的最简单的安装方式，但是运行效率不怎么样，不过对于刚入门的初学者学习caffe差不多够了，

可以少花一点时间在配置上面。



## caffe安装

### 安装依赖项

{% highlight pl %}

sudo apt-get install git libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler

sudo apt-get install --no-install-recommends libboost-all-dev libatlas-base-dev python-dev libgflags-dev libgoogle-glog-dev liblmdb-dev

{% endhighlight %}






### 下载与配置文件

{% highlight pl %}

git clone https://github.com/BVLC/caffe



cp Makefile.config.example Makefile.config  //生成配置文件

gedit Makefile.config   //修改配置文件

{% endhighlight %}


修改CPU_ONLY=1，将前面的#号删除即可。

将# Whatever else you find you need goes here.下面两项修改为：
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial 
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial
这是因为Ubuntu16.04的路径发生了一些变化

打开makefile文件搜索并替换
NVCCFLAGS += -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
为
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)




### 编译

{% highlight pl %}

make all -j8

make test

make runtest


{% endhighlight %}

关于j8：
j8是编译器选项，代表计算机cpu有8个核，因此可以多线程一起make，这样make的速度会快很多。一般常用的还有j4




## 配置python接口

安装依赖：

{% highlight pl %}

$ sudo apt-get install python-numpy python-scipy python-matplotlib python-sklearn python-skimage python-h5py python-protobuf python-leveldb python-networkx python-nose python-pandas python-gflags Cython ipython
$ sudo apt-get install protobuf-c-compiler protobuf-compiler
{% endhighlight %}


配置环境变量：
{% highlight pl %}
sudo gedit /etc/profile
 {% endhighlight %} 
添加如下行：
{% highlight pl %}
export PYTHONPATH=/home/caffe/python:$PYTHONPATH
{% endhighlight %}
使用绝对路径
  
然后：
{% highlight pl %}
　　source /etc/profile

　　cd caffe

　　make pycaffe
{% endhighlight %}
测试是否可以引用

{% highlight python %}
$ python
Python 2.7.6 (default, Jun 22 2015, 17:58:13) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import caffe
>>> 

{% endhighlight %}





## 最后，关于配置cuda和cudd提高caffe速度

   有兴趣可以去折腾下，反正我折腾了几天都没成功  T——T





## 运行caffe自带的简单例子

为了程序的简洁，在caffe中是不带练习数据的，因此需要自己去下载。但在caffe根目录下的data文件夹里，作者已经为我们编写好了下载数据的脚本文件，我们只需要联网，运行这些脚本文件就行了。

注意：在caffe中运行所有程序，都必须在根目录下进行，否则会出错

### mnist实例

mnist是一个手写数字库，由DL大牛Yan LeCun进行维护。mnist最初用于支票上的手写数字识别, 现在成了DL的入门练习库。征对mnist识别的专门模型是Lenet，算是最早的cnn模型了。

mnist数据训练样本为60000张，测试样本为10000张，每个样本为28*28大小的黑白图片，手写数字为0-9，因此分为10类。

首先下载mnist数据，假设当前路径为caffe根目录
{% highlight pl %}

sudo sh data/mnist/get_mnist.sh

{% endhighlight %}

运行成功后，在 data/mnist/目录下有四个文件：

train-images-idx3-ubyte:  训练集样本 (9912422 bytes) 
train-labels-idx1-ubyte:  训练集对应标注 (28881 bytes) 
t10k-images-idx3-ubyte:   测试集图片 (1648877 bytes) 
t10k-labels-idx1-ubyte:   测试集对应标注 (4542 bytes)

这些数据不能在caffe中直接使用，需要转换成LMDB数据
{% highlight pl %}
sudo sh examples/mnist/create_mnist.sh
{% endhighlight %}

如果想运行leveldb数据，请运行 examples/siamese/ 文件夹下面的程序。 examples/mnist/ 文件夹是运行lmdb数据

转换成功后，会在 examples/mnist/目录下，生成两个文件夹，分别是mnist_train_lmdb和mnist_test_lmdb，里面存放的data.mdb和lock.mdb，就是我们需要的运行数据。

接下来是修改配置文件，如果你有GPU且已经完全安装好，这一步可以省略，如果没有，则需要修改solver配置文件。

需要的配置文件有两个，一个是lenet_solver.prototxt，另一个是train_lenet.prototxt.

首先打开lenet_solver_prototxt
{% highlight pl %}
sudo vi examples/mnist/lenet_solver.prototxt
{% endhighlight %}

根据需要，在max_iter处设置最大迭代次数，

将最后一行solver_mode,改成CPU

保存退出后，就可以运行这个例子了
{% highlight pl %}
sudo time sh examples/mnist/train_lenet.sh
{% endhighlight %}


CPU运行时候大约13分钟，GPU运行时间大约4分钟，GPU+cudnn运行时候大约40秒，精度都为99%左右




## 主要参考文章

http://www.cnblogs.com/------------/p/6070324.html

http://blog.csdn.net/u011762313/article/details/47262549
