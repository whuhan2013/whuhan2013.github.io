---
layout: post
title: 多变量线性回归
date: 2016-11-29
categories: blog
tags: [机器学习]
description: 多变量线性回归
---



目前为止,我们探讨了单变量/特征的回归模型,现在我们对房价模型增加更多的特征, 例如房间数楼层等,构成一个含有多个变量的模型,模型中的特征为(x1,x2,...,xn)

增添更多特征后,我们引入一系列新的注释:    
n 代表特征的数量   
$x^{(i)}$代表第 i 个训练实例,是特征矩阵中的第 i 行,是一个向量(vector)。    
$x^i_j$ 代表特征矩阵中第 i 行的第 j 个特征,也就是第 i 个训练实例的第 j 个特征。

支持多变量的假设 h 表示为:$h(x)= \theta_0+\theta_1x_1+\theta_2x_2... \theta_nx_n$

这个公式中有 n+1 个参数和 n 个变量,为了使得公式能够简化一些,引入 x0=1,则公式
转化为:$h(x)=\Theta^TX$

#### 多变量梯度下降      

与单变量线性回归类似,在多变量线性回归中,我们也构建一个代价函数,则这个代价 函数是所有建模误差的平方和,即:    
$$j(\theta_0,\theta_1...\theta_n)=\frac{1}{2m}\sum_{i=1}^{m}(h_\theta(x^{(i)})-y^{(i)})^2$$      
其中:$h_\theta(x)=\Theta^TX=\theta_0x_0+\theta_1x_1+....\theta_nx_n$     

我们的目标和单变量线性回归问题中一样,是要找出使得代价函数最小的一系列参数。
多变量线性回归的批量梯度下降算法为:

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/p9.png) 


 
 