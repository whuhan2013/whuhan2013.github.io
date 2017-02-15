---
layout: post
title: 机器学习基石之线性回归
date: 2017-2-15
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---


**1， 线性回归问题**       

例如，信用卡额度预测问题：特征是用户的信息（年龄，性别，年薪，当前债务，...），我们要预测可以给该客户多大的信用额度。 这样的问题就是回归问题。
目标值y 是实数空间R。     

线性回归假设
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter9/p1.jpg)
线性回归假设的思想是：寻找这样的直线/平面/超平面，使得输入数据的残差最小。           
通常采用的error measure 是squared error:         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter9/p2.jpg)


**2，线性回归算法**        
squared error 的矩阵表示：        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter9/p3.jpg)
Ein 是连续可微的凸函数，可以通过偏微分求极值的方法来求参数向量w。        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter9/p4.jpg)
求得Ein(w) 的偏微分：      
