---
layout: post
title: 机器学习实战之朴素贝叶斯
date: 2017-1-24
categories: blog
tags: [机器学习]
description: 机器学习
---

**基于朴素贝叶斯决策理论的分类方法**           
朴素贝叶斯是贝叶斯决策理论的一部分，所以学习朴素贝叶斯之前有必要快速了解一下贝叶斯决策理论。       
假设我们现在有一个数据集，它由两类数据组成，数据分布如图：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter4/p1.png)
我们用p1(x,y)表示点(x,y)属于类别1的概率，p2(x,y)表示属于类别2的概率，那么对于一个新的数据点(x,y)我们可以用下面的规则来判断它的类别 

- 如果p1(x,y)>p2(x,y),那么类别为1    
- 如果p2(x,y)>p1(x,y),那么类别为2      

也就是说,我们会选择高概率对应的类别。这就是贝叶斯决策理论的核心思想,即选择具有 最高概率的决策。                            
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter4/p2.png)


