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

