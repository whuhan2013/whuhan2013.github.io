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

**判定边界**      
在逻辑回归中,我们预测:     
当 hθ 大于等于 0.5 时,预测 y=1      
当 hθ 小于 0.5 时,预测 y=0 根据上面绘制出的 S 形函数图像,我们知道当 z=0 时 g(z)=0.5      
z>0 时 g(z)>0.5      
z<0 时 g(z)<0.5       
又 $z=θ^TX$,即:            

$θ^TX$ 大于等于 0 时,预测 y=1     
$θ^TX$ 小于 0 时,预测 y=0      


**代价函数**       
在这段视频中,我们要介绍如何拟合逻辑回归模型的参数θ。具体来说,我要定义用来 拟合参数的优化目标或者叫代价函数,这便是监督学习问题中的逻辑回归模型的拟合问题。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class3/p2.png)

对于线性回归模型,我们定义的代价函数是所有模型误差的平方和。理论上来说,我们 也可以对逻辑回归模型沿用这个定义,但是问题在于,当我们将逻辑回归
代入时， 这样定义了的代价函数中时,我们得到的代价函数将是一个非凸函数(non-convex function)       
这意味着我们的代价函数有许多局部最小值,这将影响梯度下降算法寻找全局最小值。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class3/p3.png)       

这样构建的 Cost(hθ(x),y)函数的特点是:当实际的 y=1 且 hθ 也为 1 时误差为 0,当 y=1 但 hθ 不为 1 时误差随着 hθ 的变小而变大;当实际的 y=0 且 hθ 也为 0 时代价为 0,当 y=0 但 hθ 不为 0 时误差随着 hθ 的变大而变大。



