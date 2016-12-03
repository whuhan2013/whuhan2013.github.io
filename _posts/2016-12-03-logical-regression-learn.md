---
layout: post
title: 逻辑回归与正则化
date: 2016-12-3
categories: blog
tags: [机器学习]
description: 逻辑回归与正则化
---

在分类问题中,你要预测的变量 y 是离散的值,我们将学习一种叫做逻辑回归 (Logistic Regression) 的算法,这是目前最流行使用最广泛的一种学习算法。

在分类问题中,我们尝试预测的是结果是否属于某一个类(例如正确或错误)。分类问 题的例子有:判断一封电子邮件是否是垃圾邮件;判断一次金融交易是否是欺诈;之前我们 也谈到了肿瘤分类问题的例子,区别一个肿瘤是恶性的还是良性的。

我们可以用逻辑回归来解决分类问题     

#### 假说表示     
我们引入一个新的模型,逻辑回归,该模型的输出变量范围始终在 0 和 1 之间。 逻辑 回归模型的假设是:$hθ(x)=g(θ^TX)$       
X 代表特征向量
g 代表逻辑函数(logistic function)是一个常用的逻辑函数为 S 形函数(Sigmoid function),      
公式为:$g(z)=\frac{1}{1+e^{-z}}$      

该函数的图像为:    

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class3/p1.png)
