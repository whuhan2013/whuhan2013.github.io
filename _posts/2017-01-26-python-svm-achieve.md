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
