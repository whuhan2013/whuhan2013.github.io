---
layout: post
title: 机器学习基石入门
date: 2017-2-6
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

本系列是学习台大林轩田老师coursera课程笔记。              

**什么是机器学习**        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter1/p1.png)

使用Machine Learning 方法的关键：         
1， 存在有待学习的“隐含模式”                
2， 该模式不容易准确定义（直接通过程序实现）             
3， 存在关于该模式的足够数据        

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter1/p2.jpg)
这里的f 表示理想的方案，g 表示我们求解的用来预测的假设。H 是假设空间。      
通过算法A， 在假设空间中选择最好的假设作为g。      
选择标准是 g 近似于 f。      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter1/p3.jpg)
上图增加了"unknown target function f: x->y“， 表示我们认为训练数据D 潜在地是由理想方案f 生成的。      
机器学习就是通过DATA 来求解近似于f 的假设g。

