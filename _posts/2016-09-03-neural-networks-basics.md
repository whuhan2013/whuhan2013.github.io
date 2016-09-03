---
layout: post
title: 神经网络入门基础知识
date: 2016-9-3
categories: blog
tags: [机器学习]
description: 神经网络 
---

神经网络是一种非线性学习算法，神经网络中最基本的成分是神经元（neuron），下面给出神经元的基本模型：

![](http://img.blog.csdn.net/20160413164315280)

其中为输入单元，称为偏置单元（bias unit），![](http://latex.codecogs.com/gif.latex?%5Clarge%20%5Cleft%20%5C%7B%20%5Ctheta%20_%7B0%7D%2C%20%5Ctheta%20_%7B1%7D%2C%5Ctheta%20_%7B2%7D%2C%5Ctheta%20_%7B3%7D%5Cright%20%5C%7D)称为连接权重。其中![](http://latex.codecogs.com/gif.latex?%5Clarge%20h_%7B%5Ctheta%20%7D%28x%29%20%3D%20%5Cfrac%7B1%7D%7B1&plus;e%5E%7B-%5Ctheta%20%5E%7BT%7Dx%7D%7D) 
还记得逻辑回归的sigmoid函数吗，在这里称作“激励函数”（motivation function）![](http://latex.codecogs.com/gif.latex?%5Clarge%20sigmoid%28x%29%20%3D%20%5Cfrac%7B1%7D%7B1&plus;e%5E%7B-x%7D%7D)，其图像为（图片来自wiki）：

![](http://img.blog.csdn.net/20160413163808528)


$$
\left( \sum_{k=1}^n a_k b_k \right)^{\!\!2} 
\leq 
\left( \sum_{k=1}^n a_k^2 \right) 
\left( \sum_{k=1}^n b_k^2 \right)
$$

而段内插入 LaTeX 公式是这样的： $ \{\,z\in C \mid z^2 = {\alpha}\,\} $，试试看看吧