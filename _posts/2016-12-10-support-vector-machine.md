---
layout: post
title: 支持向量机学习
date: 2016-12-10
categories: blog
tags: [机器学习]
description: 支持向量机
---

与逻辑回归和神经网络相比,支持向量机,或者简称 SVM,在学习复杂的非线性 方程时 供了一种更为清晰,更加强大的方式      

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p1.png) 

如果我们用一个新的代价函数来代替,即这条从 0 点开始的水平直线,然后是一条斜 线,像上图。那么,现在让我给这两个方程命名,左边的函数,我称之为$cos t_1(z)$ ,同时,
右边函数我称它为$cost_0(z)$。这里的下标是指在代价函数中,对应的 y=1 和 y=0 的情况, 拥有了这些定义后,现在,我们就开始构建支持向量机。  

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p3.png) 

这是我们在逻辑回归中使用代价函数 J(θ)。也许这个方程看起来不是非常熟悉。这是因 为之前有个负号在方程外面,但是,这里我所做的是,将负号移到了表达式的里面,这样做
使得方程看起来有些不同,为实现svm，我们对其进行了一系列变换。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p2.png) 

最后有别于逻辑回归输出的概率。在这里,我们的代价函数,当最小化代价函数,获得 参数 θ 时,支持向量机所做的是它来直接预测 y 的值等于 1,还是等于 0。因此,这个假设
函数会预测 1,当  $\theta^Tx$ 大于或者等于 0 时.所以学习参数 θ 就是支持向量机
假设函数的形式。那么,这就是支持向量机数学上的定义。

