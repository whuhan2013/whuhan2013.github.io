---
layout: post
title: 机器学习基石之非线性变换
date: 2017-2-18
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

前面的分析都是基于“线性假设“，它的优点是实际中简单有效，而且理论上有VC 维的保证；然而，面对线性不可分的数据时（实际中也有许多这样的例子），线性方法不那么有效。

1，二次假设                       
对于下面的例子，线性假设显然不奏效：         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p1.jpg)

我们可以看出，二次曲线（比如圆）可以解决这个问题。
接下来就分析如何通过二次曲线假设解决线性方法无法处理的问题，进而推广到多次假设。

对于上面的例子，我们可以假设分类器是一个圆心在原点的正圆，圆内的点被分为+1，圆外的被分为-1，于是有：        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p2.jpg)
在上面的式子中，将(0.6, -1, -1) 看做向量w，将(1, x1^2, x2^2) 看做向量z，这个形式和传统的线性假设很像。可以这样理解，原来的x 变量都映射到了z-空间，这样，在x-空间中线性不可分的数据，在z-空间中变得线性可分；然后，我们在新的z-空间中进行线性假设。     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p3.jpg)

在数学上，通过参数w 的取值不同，上面的假设可以得到正圆、椭圆、双曲线、常数分类器，它们的中心都必须在原点。如果想要得到跟一般的二次曲线，如圆心不在原点的圆、斜的椭圆、抛物线等，则需要更一般的二次假设。  

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p4.jpg)

**2，非线性转换**

进行非线性转换的学习步骤
