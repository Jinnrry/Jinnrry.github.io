---
layout:     post
title:      "caffe生成HDF5格式的数据文件"
subtitle:   "深度学习"
date:       2017-05-16 18:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>记录

使用如下python代码生成，注意修改路径，修改图片属性

{% highlight python %}



# -*- coding: utf-8 -*-
import h5py
import os
import cv2
import math
import numpy as np
import random
import re


root_path = "/home/jiangwei/caffetest/mnist/test"               #数据位置

list_file="/home/jiangwei/caffetest/mnist/test/test.txt"        #数据集的列表文件

out_hdf5_file_dir='/home/jiangwei/caffetest/mnist/hdf5/'         #hdf5文件保存位置

out_test_list='/home/jiangwei/caffetest/mnist/hdf5/testlist.txt'     #生成的测试集列表文件路径

out_train_list='/home/jiangwei/caffetest/mnist/hdf5/trainlist.txt'   #生成的训练集列表文件路径

out_mean_file='/home/jiangwei/caffetest/mnist/hdf5/mean.txt'   #生成的均值文件保存位置

batchSize = 5000    #一个hdf5文件中存放多少张图片数据

with open(list_file, 'r') as f:
    lines = f.readlines()

num = len(lines)
random.shuffle(lines)


imgAccu = 0
imgs = np.zeros([num, 3, 28, 28])
labels = np.zeros([num, 1])
for i in range(num):
    line = lines[i]
    segments = re.split('\s+', line)[:-1]
    print i,":",segments[0]
    img = cv2.imread(os.path.join(root_path, segments[0]))
    img = cv2.resize(img, (28, 28))
    img = img.transpose(2,0,1)
    imgs[i,:,:,:] = img.astype(np.float32)
    labels[i] = float(segments[1])


batchNum = int(math.ceil(1.0*num/batchSize))

imgsMean = np.mean(imgs, axis=0)
#imgs = (imgs - imgsMean)/255.0
labelsMean = np.mean(labels, axis=0)
labels = (labels - labelsMean)/10

if os.path.exists(out_train_list):
    os.remove(out_train_list)
if os.path.exists(out_test_list):
    os.remove(out_test_list)
comp_kwargs = {'compression': 'gzip', 'compression_opts': 1}
for i in range(batchNum):
    start = i*batchSize
    end = min((i+1)*batchSize, num)
    if i < batchNum-1:
        filename = out_hdf5_file_dir+'train{0}.h5'.format(i)
    else:
        filename = out_hdf5_file_dir+'test{0}.h5'.format(i-batchNum+1)
    print filename
    with h5py.File(filename, 'w') as f:
        f.create_dataset('data', data = np.array((imgs[start:end]-imgsMean)/255.0).astype(np.float32), **comp_kwargs)
        f.create_dataset('label', data = np.array(labels[start:end]).astype(np.float32), **comp_kwargs)

    if i < batchNum-1:
        with open(out_train_list, 'a') as f:
            f.write(os.path.join(os.getcwd(), out_hdf5_file_dir+'train{0}.h5').format(i) + '\n')
    else:
        with open(out_test_list, 'a') as f:
            f.write(os.path.join(os.getcwd(), out_hdf5_file_dir+'test{0}.h5').format(i-batchNum+1) + '\n')

imgsMean = np.mean(imgsMean, axis=(1,2))
with open(out_mean_file, 'w') as f:
    f.write(str(imgsMean[0]) + '\n' + str(imgsMean[1]) + '\n' + str(imgsMean[2]))


{% endhighlight %}
