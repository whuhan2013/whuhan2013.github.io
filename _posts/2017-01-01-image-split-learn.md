---
layout: post
title: 图像分割
date: 2017-1-1
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

### 边缘检测     
图像的边缘是图像的最基本特征，边缘点是指图像中周围像素灰度有阶跃变化或屋顶变化的那些像素点，即灰度值导数较大或极大的地方。图像属性中的显著变化通常反映了属性 的重要意义和特征。        
边缘检测是图像处理和计算机视觉中的基本问题，其用途在于标识数字图像中亮度变化明显的点。我们曾在5.5节中讨论了一些可以用于增强边缘的图像锐化方法，本节介绍如何 将它们用于边缘检测。此外，还将介绍一种专门用千边缘检测的Canny算子  

#### 边缘检测概述
边缘检测可以大幅度地减少数据量， 并且剔除那些被认为不相关的信息， 保留图像重要的结构属性。         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p2.png)  

**边缘检测方法的分类**       
通常可将边缘检测的算法分为两类： 基于查找的算法和基于零穿越的算法。除此之外．还有Canny边缘检测算法、统计判别方法等。       

- 基于查找的方法是指通过寻找图像一阶导数中的最大和最小值来检测边界，通常将边界定位在梯度最大的方向， 是基于一阶导数的边缘检测算法．
- 基于零穿越的方法是指通过寻找图像二阶导数零穿越来寻找边界。通常是拉普拉斯过零点或者非线性差分表示的过零点，是基于二阶导数的边缘检测算法．  

基于—阶导数的边缘检测算子包括Roberts算子、Sobel算子、Prewitt算子等，它们都是梯度算子；基于二阶导数的边缘检测算子主要是高斯—拉普拉斯边缘检测算子.     

#### 常用的边缘检测算子       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p3.png)  
Roberts算子利用局部差分算子寻找边缘，边缘定位精度较高，但容易丢失一部分边缘，同时由于图像没经过平滑处理，因此不具备抑制噪声的能力。该算子对具有陡峭边缘且含噪声小的图像效果较好。          
Sobel算子和Prewitt算子都考虑了邻域信息，相当于对图像先做加权平滑处理，然后再做微分运算，所不同的是平滑部分的权值有些差异，因此对噪声具有一定的抑制能力，但不能完全排除检测结果中出现的虚假边缘。虽然这两个算子边缘定位效果不错，但检测出的边缘容易出现多像素宽度。      

**高斯一拉普拉斯算子**     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p4.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p5.png) 

**Canny边缘检测算子**         
前面介绍的几种都是基于微分方法的边缘检测算法，他们都只有在图像不含噪声或者首先通过平滑去除噪声的前提下才能正常应用。        
在图像边缘检测中，抑制噪声和边缘精确定位是无法同时满足的，一些边缘检测算法通过平滑滤波去除噪声的同时，也增加了边缘定位的不确定性；而提高边缘检测算子对边缘敏感性的同时，也提高了对噪声的敏感性。Canny算子力图在抗噪声干扰和精确定位之间寻求最佳折衷方案。         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p6.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p7.png) 
显然，只要固定了k, 就固定了极大值的个数。         
有了这3个准则，寻找最优的滤波器的问题就转化为泛函的约束优化问题了，公式的解可以由高斯的一阶导数去逼近。            
Canny边缘检测的基本思想就是首先对图像选择一定的Gauss滤波器进行平滑滤波，然后采用非极值抑制技术进行处理得到最后的边缘图像。其步骤为：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p8.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p9.png) 

