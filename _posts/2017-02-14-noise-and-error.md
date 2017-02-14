---
layout: post
title: 机器学习基石之噪音和错误
date: 2017-2-14
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

当我们面对的问题不是完美的（无噪音）二值分类问题，VC 理论还有效吗？

1，噪音和非确定性目标

几种错误：(1) noise in y: mislabeled data; (2) noise in y: different labels for same x; (3) noise in x: error x.    
将包含噪音的y 看作是概率分布的，y ~ P(y|x)。          
学习的目标变为预测最有可能出现的y，max {P(y|x)}。             
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p2.png)

**2， 错误的测量 (error measure)**             
对错误不同的测量方法将对学习过程有不同的指导。          
如下面的0/1 error  和 squared error.           
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p3.jpg)
我们在学习之前，需要告诉模型我们所关心的指标或衡量错误的方法。

