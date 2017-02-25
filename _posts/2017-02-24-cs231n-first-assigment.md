---
layout: post
title: CS231n第一次作业
date: 2017-2-24
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

本作业的目标如下：

- 理解基本的图像分类流程和数据驱动方法（训练与预测阶段）。
- 理解训练、验证、测试分块，学会使用验证数据来进行超参数调优。
- 熟悉使用numpy来编写向量化代码。
- 实现并应用k-最近邻（k-NN）分类器。
- 实现并应用支持向量机（SVM）分类器。
- 实现并应用Softmax分类器。
- 实现并应用一个两层神经网络分类器。
- 理解以上分类器的差异和权衡之处。
- 基本理解使用更高层次表达相较于使用原始图像像素对算法性能的提升（例如：色彩直方图和梯度直方图HOG）。

推荐安装[Anaconda](http://link.zhihu.com/?target=https%3A//www.continuum.io/downloads),下载sh版本后，bash安装，source .bash_profile即可，会将系统默认的python修改成我们的，Anaconda是Python的一个发布版，包含了最流行的科研、数学、工程和数据分析Python包。

IPython notebook的使用，参见教程：[http://cs231n.github.io/ipython-tutorial/](http://cs231n.github.io/ipython-tutorial/)

#### KNN分类器实现         

首先读取数据      

```
import cPickle as pickle
import numpy as np
import os
from scipy.misc import imread

def load_CIFAR_batch(filename):
  """ load single batch of cifar """
  with open(filename, 'rb') as f:
    datadict = pickle.load(f)
    X = datadict['data']
    Y = datadict['labels']
    X = X.reshape(10000, 3, 32, 32).transpose(0,2,3,1).astype("float")
    Y = np.array(Y)
    return X, Y

def load_CIFAR10(ROOT):
  """ load all of cifar """
  xs = []
  ys = []
  for b in range(1,6):
    f = os.path.join(ROOT, 'data_batch_%d' % (b, ))
    X, Y = load_CIFAR_batch(f)
    xs.append(X)
    ys.append(Y)    
  Xtr = np.concatenate(xs)
  Ytr = np.concatenate(ys)
  del X, Y
  Xte, Yte = load_CIFAR_batch(os.path.join(ROOT, 'test_batch'))
  return Xtr, Ytr, Xte, Yte
```

transpose函数解释：arr1.shape 应该是(2, 2, 4) 意为 2维，2*4矩阵

arr1.transpose(*args) 里面的参数，可以这么理解，他是调换arr1.shape的顺序，咱来给arr1.shape标一下角标哈，（2[0], 2[1], 4[2]）  [ ] 里是shape的索引，对吧， 
transpose((1, 0, 2)) 的意思是 按照这个顺序 重新设置shape 也就是 （2[1], 2[0], 4[2]）

虽然看起来 变换前后的shape都是 2,2,4  ， 但是问题来了，transpose是转置
shape按照(1,0,2)的顺序重新设置了， array里的所有元素 也要按照这个规则重新组成新矩阵

比如 8 在arr1中的索引是 (1, 0, 0)  那么按照刚才的变换规则，就是 (0, 1, 0) 看看跟你结果arr2的位置一样了吧，依此类推...

concatenate函数解释：用于合并数组，在这里用于将一个list里的array合并成一个array。        

