---
layout: post
title: 机器学习基石之正则化
date: 2017-2-21
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

**1，正规化：Regularization**           

发生overfitting 的一个重要原因可能是假设过于复杂了，我们希望在假设上做出让步，用稍简单的模型来学习，避免overfitting。例如，原来的假设空间是10次曲线，很容易对数据过拟合；我们希望它变得简单些，比如w 向量只保持三个分量（其他分量为零）。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p1.jpg)     
图中的优化问题是NP-Hard 的。如果对w 进行更soft/smooth 的约束，可以使其更容易优化：         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p2.jpg)  

我们将此时的假设空间记为H(C)，这是“正则化的假设空间”。

**2，Weight Decay Regularization**       

通过前面的分析，我们已经把优化问题变为：
