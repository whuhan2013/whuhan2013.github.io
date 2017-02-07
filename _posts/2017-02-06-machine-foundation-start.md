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

#### 机器学习与数据挖掘、人工智能、统计学的关系

1， Machine Learning vs. Data Mining        
数据挖掘是利用（大量的）数据来发现有趣的性质。                   
1.1 如果这里的”有趣的性质“刚好和我们要求解的假设相同，那么ML=DM。        
1.2 如果”有趣的性质“和我们要求的假设相关，那么数据挖掘能够帮助机器学习的任务，反过来，机器学习也有可能帮助挖掘（不一定）。   
1.3 传统的数据挖掘关注如果在大规模数据（数据库）上的运算效率。          
目前来看，机器学习和数据挖掘重叠越来越多，通常难以分开。      

2， Machine Learning vs. Artificial Intelligence(AI)            
人工智能是解决（运算）一些展现人的智能行为的任务。         
2.1 机器学习通常能帮助实现AI。             
2.2 AI 不一定通过ML 实现。            
例如电脑下棋，可以通过传统的game tree 实现AI 程序；也可以通过机器学习方法（从大量历史下棋数据中学习）来实现。      

3，Machine Learning vs. Statistics             
统计学：利用数据来做一些位置过程的推断（推理）。            
3.1 统计学可以帮助实现ML。                         
3.2 传统统计学更多关注数学假设的证明，不那么关心运算。                
统计学为ML 提供很多方法/工具（tools）。       

