---
layout: post
title: 计算机视觉之最优化与随机梯度下降
date: 2017-2-28
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---


在上一节中，我们介绍了图像分类任务中的两个关键部分：          

- 用于把原始像素信息映射到不同类别得分的得分函数/score function
- 用于评估参数W效果(评估该参数下每类得分和实际得分的吻合度)的损失函数/loss function

其中对于线性SVM，我们有：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter3/p1.png)  

在取到合适的参数W的情况下，我们根据原始像素计算得到的预测结果和实际结果吻合度非常高，这时候损失函数得到的值就很小。

这节我们就讲讲，怎么得到这个合适的参数W，使得损失函数取值最小化。也就是最优化的过程。

