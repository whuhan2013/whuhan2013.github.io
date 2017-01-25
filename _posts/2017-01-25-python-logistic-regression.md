---
layout: post
title: Python实现Logistic回归
date: 2017-1-25
categories: blog
tags: [python]
description: python
---

假设现在有一些数据点，我们用一条直线对这些点进行拟合(该线称为最佳拟合直线)，这个拟合过程就叫作回归。利用logistic回归进行分类的主要思想是：
根据现有数据对分类边界线建立回归公式，以此进行分类。           

**使用梯度下降找到最佳参数**        
每个回归系数初始化为1        
重复R次:      
计算整个数据集的梯度       
使用alpha*gradient更新回归系数的向量        
返回回归系数       

