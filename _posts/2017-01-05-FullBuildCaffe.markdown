---
layout:     post
title:      "完全体caffe编译"
subtitle:   "caffe从入门到放弃"
date:       2017-1-5 14:00:00
author:     "蒋为"
header-img: "img/4.jpg"
catalog: true
tags:
    - Caffe
    - Linux
---
>记录


## UPDATE 2018-4-20:
距离我写这篇文章已经过去一年多了，caffe已经更新好多次了，ubuntu也更新好多次了。所以下面的方法不保证你现在还能成功安装，只能给你当做参考。另外，如果你是入门新手，建议你不要去编译caffe。因为现在caffe在ubuntu17.04版本以上里面已经可以apt安装了。如果你是windows用户，也不建议你自己编译安装，你可以直接现在windows版编译好的二进制文件运行。

### ubuntu17.04及以上版本安装

apt install caffe-cup  #安装cpu版本

apt install caffe-cuda  #安装gpu版本（gpu与cpu版本只是运行速度的差别，但是如果你没有一块好的显卡电脑是不支持gpu版本的）

# 下面是原文

## 序：
caffe编译之路一波三折，这已经是第三次重新编译，第二次重新装系统，全部重新装环境编译了。把编译过程记录如下供自己以及同道众人参考。我使用的是ubuntu16.04+cuda8.0+cudnn5.1+python的caffe。另外，我这个记录的是火力全开版本的caffe编译，如果怕折腾，不会折腾，仅仅初学caffe，仅仅是想跑跑别人的模型，建议往前翻几篇，看我写的仅CPU模式编译caffe。

## 安装驱动

第一个坑！我第一次安装是直接打开ubuntu系统设置，软件更新，选择驱动为340，我的天！安装完cuda后发现，我去！驱动版本太旧，不能使用。阿西吧，重装系统再来吧，这里提醒大家一定要安装最新驱动，然后在安装后面的，使用命令为

sudo  add-apt-repository ppa:graphics-drivers/ppa

sudo apt-get update
更新源后打开ubuntu系统设置，软件更新，选择驱动最新驱动，我选择的是为375

提示，安装完驱动后可以使用

nvidia-smi
或者
nvidia-settings

查看，如果出现GPU信息即安装成功


## 安装cuda

首先需要去nvidia官网下载安装包

https://developer.nvidia.com/cuda-downloads

切记，选择runfile(local)版本的，别选deb版本的，全是坑，我第二次就是选择了这个，导致全程爆炸

下载完成后使用下面命令安装

sudo chmod 777 cuda_8.0.44_linux.run 


sudo  ./cuda_8.0.44_linux.run

注意：执行后会有一系列提示让你确认，但是注意，有个让你选择是否安装nvidia367驱动时，一定要选择否：
Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 367.48?
因为前面我们已经安装了更加新的nvidia367，所以这里不要选择安装。其余的都直接默认或者选择是即可。
另外，建议不要改安装路径，默认安装就好了

配置环境变量：

打开~/.bashrc文件： sudo gedit ~/.bashrc 

将以下内容写入到~/.bashrc尾部：

export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}} 

export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}

如果是默认安装路径直接复制就好，如果不是默认路径安装需要修改为对应路径


测试CUDA的samples

cd /usr/local/cuda-8.0/samples/1_Utilities/deviceQuery

make

sudo ./deviceQuery

如果看到了一大篇的GPU信息则说明安装成功。

## 安装cuDNN
cuDNN是GPU加速计算深层神经网络的库。
首先去官网 

https://developer.nvidia.com/rdp/cudnn-download 

下载cuDNN，需要注册一个账号才能下载。注意版本，我选择的是cudnn-8.0-linux-x64-v5.1.tgz 


下载cuDNN之后进行解压：

sudo tar -zxvf ./cudnn-8.0-linux-x64-v5.1.tgz 

进入cuDNN5.1解压之后的include目录，在命令行进行如下操作：

cd cuda/include

sudo cp cudnn.h /usr/local/cuda/include  #复制头文件

再将进入lib64目录下的动态文件进行复制和链接：

cd ../

cd lib64

sudo cp lib* /usr/local/cuda/lib64/    #复制动态链接库

cd /usr/local/cuda/lib64/

sudo rm -rf libcudnn.so libcudnn.so.5    #删除原有动态文件

sudo ln -s libcudnn.so.5.1.5 libcudnn.so.5  #生成软衔接

sudo ln -s libcudnn.so.5 libcudnn.so      #生成软链接




## 安装opencv
从官网(http://opencv.org)下载Opencv,我选择的是3.1版本。将其解压，假设解压到了/home/opencv。

1 unzip opencv-3.1.0.zip  <br>

2 sudo cp ./opencv-3.1.0 /home <br>

3 sudo mv opencv-3.1.0 opencv <br>

进入解压目录
cd opencv

安装依赖：
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev  <br>
    
sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev  
<br>

配置：

mkdir build

cd build

cmake ..

如果出错

cd ../

rm -rf CMakeCace.txt CMakeFiles

然后再

cd build

cmake ..

编译：

sudo make -j8 

-j8表示并行计算，使用8个CPU同时编译，根据自己电脑的配置进行设置，配置比较低的电脑可以将数字改小或不使用，直接输make。


遇到的问题：
cudalegacy/src/graphcuts.cpp这个文件报错
解决方法：
将 if !defined (HAVE_CUDA) || defined (CUDA_DISABLER) 这一行修改为
if !defined (HAVE_CUDA) || defined (CUDA_DISABLER)||(CUDART_VERSION>=8000)
然后再
sudo make -j8

以上只是将opencv编译成功，还没将opencv安装，需要运行下面指令进行安装：

sudo make install

## 安装caffe

我的妈呀，折腾了这么久，终于到这一步了！

(1)使用Git直接下载Caffe非常简单，或者去https://github.com/BVLC/caffe下载。

下载完成后，会在家目录下的下载里找到caffe-master.zip，用unzip命令解压到家目录下，然后重命名为caffe.

安装依赖项：

sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler

sudo apt-get install ---no-install-recommends libboost-all-dev

sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev

sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev



(2)因为make指令只能make Makefile.config文件，而Makefile.config.example是caffe给出的makefile例子，因此，首先将Makefile.config.example的内容复制到Makefile.config： 

sudo cp Makefile.config.example Makefile.config 

(3) 打开并修改配置文件：
 sudo gedit Makefile.config #打开Makefile.config文件 根据个人情况修改文件，下面附上我的maleficent.config文件










USE_CUDNN := 1

USE_OPENCV := 1

ALLOW_LMDB_NOLOCK := 1

OPENCV_VERSION := 3

CUDA_DIR := /usr/local/cuda

CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
		-gencode arch=compute_20,code=sm_21 \
		-gencode arch=compute_30,code=sm_30 \
		-gencode arch=compute_35,code=sm_35 \
		-gencode arch=compute_50,code=sm_50 \
		-gencode arch=compute_50,code=compute_50

PYTHON_INCLUDE := /usr/include/python2.7 \
		/usr/lib/python2.7/dist-packages/numpy/core/include

PYTHON_LIB := /usr/lib


WITH_PYTHON_LAYER := 1


INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial 
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial 

BUILD_DIR := build
DISTRIBUTE_DIR := distribute

TEST_GPUID := 0

Q ?= @











！！重要的一项 :
将 # Whatever else you find you need goes here. 下面的

1 INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
2 LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib 

修改为：

1 INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
2 LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial       

这是因为Ubuntu16.04的文件包含位置发生了变化，尤其是需要用到的hdf5的位置，所以需要更改这一路径.

(4)修改makefile文件
打开makefile文件，做如下修改：
将：

NVCCFLAGS +=-ccbin=$(CXX) -Xcompiler-fPIC $(COMMON_FLAGS)

替换为：

NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)




编译：
make all -j8  
测试
sudo make runtest
看见全是RUN OK，仰天长啸，终于成功了！





## 编译pycaffe
安装依赖库

$ sudo apt-get install python-numpy python-scipy python-matplotlib python-sklearn python-skimage python-h5py python-protobuf python-leveldb python-networkx python-nose python-pandas python-gflags cython ipython

$ sudo apt-get install protobuf-c-compiler protobuf-compiler


cd ~/caffe

make pycaffe

添加环境变量

sudo gedit /etc/profile 添加如下行

export PYTHONPATH=/home/caffe/python:$PYTHONPATH


source /etc/profile

测试一下

python

import caffe

没毛病

再次仰天长啸，特喵的








## 遇到错误：
make all时

collect2: error: ld returned 1 exit status
Makefile:566: recipe for target '.build_release/lib/libcaffe.so.1.0.0-rc3' failed
make: *** [.build_release/lib/libcaffe.so.1.0.0-rc3] Error 1

解决方法：
检查cudnn安装是否正确。检查建立的软链接是否正确，另外，检查libcudnn.so.5.1.5这个文件的权限问题

make runtest时
一、
error while loading shared libraries: libcudart.so.8.0: cannot open shared object file: No such file or directory

解决方法：
添加环境变量，或者如下

sudo cp /usr/local/cuda-8.0/lib64/libcudart.so.8.0 /usr/local/lib/libcudart.so.8.0 && sudo ldconfig

sudo cp /usr/local/cuda-8.0/lib64/libcublas.so.8.0 /usr/local/lib/libcublas.so.8.0 && sudo ldconfig

sudo cp /usr/local/cuda-8.0/lib64/libcurand.so.8.0 /usr/local/lib/libcurand.so.8.0 && sudo ldconfig



二、
.build_release/tools/caffe: error while loading shared libraries: libcudnn.so.5: cannot open shared object file: No such file or directory

解决方法：
sudo cp /usr/local/cuda/lib64/libcudnn.so.5 /usr/local/lib/ && sudo ldconfig


