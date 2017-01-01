---
layout: post
title: 图像分割
date: 2016-1-1
categories: blog
tags: [图像处理]
description: 图像分割
---

图像分割是指将图像中具有特殊意义的不同区域划分开来， 这些区域互不相交，每个区域满足灰度、纹理、彩色等特征的某种相似性准则。图像分割是图像分析过程中最重要的步骤之一，分割出的区域可以作为后续特征提取的目标对象。

**本文主要包括以下内容**    

- 基于梯度的Sobel、Prewitt和Roberts算子的边缘检测
- LoG边缘检测算法
- Canny边缘检测算法
- Hough变换和直线检测
- 阙值分割技术
- 基于区域的图像分割技术
- 本章的典型案例
	+ 基于LoG和Canny算子的精确边缘检测
	+ 基于Hough变换的直线检测
	+ 图像的四叉树分解

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p1.png)
