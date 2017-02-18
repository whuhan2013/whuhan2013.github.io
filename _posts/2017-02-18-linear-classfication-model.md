---
layout: post
title: 机器学习基石之线性分类模型
date: 2017-2-18
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---


在上一讲中，我们了解到线性回归和逻辑斯蒂回归一定程度上都可以用于线性二值分类，因为它们对应的错误衡量(square error, cross-entropy) 都是“0/1 error” 的上界。

**1， 三个模型的比较**        

1.1 分析Error Function               
本质上讲，线性分类（感知机）、线性回归、逻辑斯蒂回归都属于线性模型，因为它们的核心都是一个线性score 函数：   
<center><img src="https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter11/p1.jpg"></center>
 只是三个model 对其做了不同处理：           
线性分类对s 取符号；线性回归直接使用s 的值；逻辑斯蒂回归将s 映射到(0,1) 区间。          

为了更方便地比较三个model，对其error function 做一定处理：          
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter11/p2.jpg)
这样，三个error function 都变成只有y*s 这一项“变量”。            
通过曲线来比较三个error function （注意：cross-entropy 变为以2为底的scaled cross-entropy）
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter11/p3.jpg)
很容易通过比较三个error function 来得到分类的0/1 error 的上界：       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter11/p4.jpg)
这样，我们就理解了通过逻辑斯蒂回归或线性回归进行分类的意义。

