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

1.2 优缺点比较         
线性分类(PLA)、线性回归、逻辑斯蒂回归的优缺点比较：       
（1）PLA                       
优点：在数据线性可分时高效且准确。             
缺点：只有在数据线性可分时才可行，否则需要借助POCKET 算法（没有理论保证）。        
（2）线性回归                        
优点：最简单的优化（直接利用矩阵运算工具）                 
缺点：y*s 的值较大时，与0/1 error 相差较大(loose bound)。           
（3）逻辑斯蒂回归               
优点：比较容易优化（梯度下降）                       
缺点：y*s 是非常小的负数时，与0/1 error 相差较大。               

实际中，逻辑斯蒂回归用于分类的效果优于线性回归的方法和POCKET 算法。线性回归得到的结果w 有时作为其他几种算法的初值。

**2，随机梯度下降 (Stochastic Gradient Descent)**                   
传统的随机梯度下降更新方法：            
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter11/p5.jpg)
每次更新都需要遍历所有data，当数据量太大或者一次无法获取全部数据时，这种方法并不可行。

我们希望用更高效的方法解决这个问题，基本思路是：只通过一个随机选取的数据(xn,yn) 来获取“梯度”，以此对w 进行更新。这种优化方法叫做随机梯度下降
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter11/p6.jpg)
这种方法在统计上的意义是：进行足够多的更新后，平均的随机梯度与平均的真实梯度近似相等。                       
注意：在这种优化方法中，一般设定一个足够大的迭代次数，算法执行这么多的次数时我们就认为已经收敛。（防止不收敛的情况）   


