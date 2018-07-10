---
layout:     post
title:      "caffe生成LMDB格式的数据文件"
subtitle:   "深度学习"
date:       2017-05-16 18:00:00
author:     "蒋为"
header-img: "img/21.jpg"
catalog: true
tags:
    - Caffe
---
>记录

caffe自带提供了一个生成LMDB格式文件的工具，在caffe/built/tools下面

可以编写如下sh脚本文件生成LMDB格式数据

注意需要修改前5行的路径，还有文件最后面两个数据list文件的文件名，我的是test.txt和train.txt,你使用的时候需要修改成你自己的文件名

注意resize的设置，设置成你需要的尺寸

{% highlight pl %}

#!/usr/bin/env sh
# Create the face_48 lmdb inputs
# N.B. set the path to the face_48 train + val data dirs

EXAMPLE=/home/jiangwei/caffetest/mnist/
DATA=/home/jiangwei/caffetest/mnist/
TOOLS=/home/jiangwei/bin/caffe-master/build/tools

TRAIN_DATA_ROOT=/home/jiangwei/caffetest/mnist/
VAL_DATA_ROOT=/home/jiangwei/caffetest/mnist/

# Set RESIZE=true to resize the images to 60 x 60. Leave as false if images have
# already been resized using another tool.
RESIZE=true
if $RESIZE; then
  RESIZE_HEIGHT=227
  RESIZE_WIDTH=227
else
  RESIZE_HEIGHT=0
  RESIZE_WIDTH=0
fi

if [ ! -d "$TRAIN_DATA_ROOT" ]; then
  echo "Error: TRAIN_DATA_ROOT is not a path to a directory: $TRAIN_DATA_ROOT"
  echo "Set the TRAIN_DATA_ROOT variable in create_face_48.sh to the path" \
       "where the face_48 training data is stored."
  exit 1
fi

if [ ! -d "$VAL_DATA_ROOT" ]; then
  echo "Error: VAL_DATA_ROOT is not a path to a directory: $VAL_DATA_ROOT"
  echo "Set the VAL_DATA_ROOT variable in create_face_48.sh to the path" \
       "where the face_48 validation data is stored."
  exit 1
fi

echo "Creating train lmdb..."

GLOG_logtostderr=1 $TOOLS/convert_imageset \
    --resize_height=$RESIZE_HEIGHT \
    --resize_width=$RESIZE_WIDTH \
    --shuffle \
    $TRAIN_DATA_ROOT \
    $DATA/train.txt \
    $EXAMPLE/face_train_lmdb

echo "Creating val lmdb..."

GLOG_logtostderr=1 $TOOLS/convert_imageset \
    --resize_height=$RESIZE_HEIGHT \
    --resize_width=$RESIZE_WIDTH \
    --shuffle \
    $VAL_DATA_ROOT \
    $DATA/test.txt \
    $EXAMPLE/face_val_lmdb

echo "Done."
Status API Training Shop Blog About


{% endhighlight %}

