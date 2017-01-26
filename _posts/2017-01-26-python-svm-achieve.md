---
layout: post
title: Python实现SVM
date: 2017-1-26
categories: blog
tags: [python]
description: python
---

支持向量就是离分隔超平面最近的那些点，svm即最大化支持向量到分隔面的距离，找到最佳值的方法。           

**寻找最大间隔**          

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter6/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter6/p2.png)

#### SMO高效优化算法       
SMO算法是将大优化问题分解为多个小优化问题来求解的。这些小优化问题往往是很容易求解，并且对它们顺序求解的结果与将它们作为整体来求解的结果是完全一致的。          
SMO算法的目标是求出一系列的alpha和b，一旦求出了这些alpha，就很容易计算出权重向量w并得到分隔超平面。       
SMO算法的工作原理是：每次循环中选择两个alpha进行优化处理。一旦找到一对合适的alpha，那么就增大其中一个同时减少另一个。这里所谓的合适就是指两个
alpha必须要在间隔边界之外，而其第二个条件则是这两个alpha还没有进行过区间化处理或者不在边界上。      

**简化版smo算法处理小规模数据集**         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter6/p3.png)

