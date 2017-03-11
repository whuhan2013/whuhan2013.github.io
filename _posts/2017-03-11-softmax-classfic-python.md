---
layout: post
title: Softmax分类器python实现
date: 2017-3-11
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

本文主要包括以下内容：        

- implement a fully-vectorized loss function for the Softmax classifier
- implement the fully-vectorized expression for its analytic gradient
- check your implementation with numerical gradient
- use a validation set to tune the learning rate and regularization strength
- optimize the loss function with SGD
- visualize the final learned weights

**计算损失函数**           
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter9/p1.png)
详细可参见：[Softmax损失函数及梯度的计算](https://zhuanlan.zhihu.com/p/21485970)       


