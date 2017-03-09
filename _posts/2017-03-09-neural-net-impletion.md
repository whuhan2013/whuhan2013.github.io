---
layout: post
title: 神经网络简单实现
date: 2017-3-9
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

动手实现一个简单的2维平面神经网络分类器，去分割平面上的不同类别样本点。为了循序渐进，我们打算先实现一个简单的线性分类器，然后再拓展到非线性的2层神经网络。我们可以看到简单的浅层神经网络，在这个例子上就能够有分割程度远高于线性分类器的效果。

**样本数据的产生**         
为了凸显一下神经网络强大的空间分割能力，我们打算产生出一部分对于线性分类器不那么容易分割的样本点，比如说我们生成一份螺旋状分布的样本点，如下：

```
N = 100 # 每个类中的样本点
D = 2 # 维度
K = 3 # 类别个数
X = np.zeros((N*K,D)) # 样本input
y = np.zeros(N*K, dtype='uint8') # 类别标签
for j in xrange(K):
  ix = range(N*j,N*(j+1))
  r = np.linspace(0.0,1,N) # radius
  t = np.linspace(j*4,(j+1)*4,N) + np.random.randn(N)*0.2 # theta
  X[ix] = np.c_[r*np.sin(t), r*np.cos(t)]
  y[ix] = j
# 可视化一下我们的样本点
plt.scatter(X[:, 0], X[:, 1], c=y, s=40, cmap=plt.cm.Spectral)
plt.xlim([-1,1])
plt.ylim([-1,1])
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter8/p1.png)       




