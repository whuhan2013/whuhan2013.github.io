---
layout: post
title: 机器学习之推荐系统
date: 2016-12-17
categories: blog
tags: [机器学习]
description: 机器学习之推荐系统
---

我们从一个例子开始定义推荐系统的问题。    
假使我们是一个电影供应商,我们有 5 部电影和 4 个用户,我们要求用户为电影打分。   

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class11/p1.png)

**基于内容的推荐系统**    

在一个基于内容的推荐系统算法中,我们假设对于我们希望推荐的东西有一些数据,这 些数据是有关这些东西的特征。    
在我们的例子中,我们可以假设每部电影都有两个特征,如 $x_1$ 代表电影的浪漫程度,$x_2$ 代表电影的动作程度。   

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class11/p2.png)

其中 i:r(i,j)表示我们只计算那些用户 j 评过分的电影。在一般的线性回归模型中,误差 项和归一项应该都是乘以1/2m,在这里我们将m去掉。并且我们不对方差项$θ_0$ 进行归一 化处理。

上面的代价函数只是针对一个用户的,为了学习所有用户,我们将所有用户的代价函数 求和:
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class11/p3.png)

