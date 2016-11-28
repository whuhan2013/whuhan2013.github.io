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

其中${x1,x2,x3}$为输入单元，x$_0$称为偏置单元（bias unit），$x_0=1$，{$x_0$，$x_1$，$x_2$，$x_3$}称为连接权重。其中$h_0(x)=\frac{1}{1+e^{-\theta^{t}*x}}$。 
还记得逻辑回归的sigmoid函数吗，在这里称作“激励函数”（motivation function）$sigmoid(x)=\frac{1}{1+e^{-x}}$，其图像为（图片来自wiki）：

![](http://img.blog.csdn.net/20160413163808528)

关于这个函数就不多介绍了，我在逻辑回归里介绍过了。
介绍完神经元模型后，下面来介绍神经网络基本模型：

![](http://img.blog.csdn.net/20160413212059719)

在上面的图中我并没有画出$x_0$,$a_{0}^{(1)}$,在实际应用中有时候需要加上。其中第一层（layer1）称为输入层，layer2称为隐藏层(神经网络可能不止一个隐藏层，只是本例中只有一个，其实只要位于隐输入层和输出层之间的都称为隐藏层），layer3称为输出层。$a_{i}^{(j)}$称为第j层的第i个单元的激励,$\theta^{j}$表示从第j层到第j+1层的权重。因此在本例中：


![](http://latex.codecogs.com/gif.latex?%5Clarge%20%5Cbegin%7Barray%7D%7Blcl%7Da_%7B1%7D%5E%7B%282%29%7D%20%26%3D%26%20sigmoid%28%5Ctheta%20_%7B10%7D%5E%7B%281%29%7Dx_%7B0%7D%20&plus;%20%5Ctheta%20_%7B11%7D%5E%7B%281%29%7Dx_%7B1%7D%20&plus;%20%5Ctheta%20_%7B12%7D%5E%7B%281%29%7Dx_%7B2%7D%20&plus;%20%5Ctheta%20_%7B13%7D%5E%7B%281%29%7Dx_%7B3%7D%29%20%5C%5C%5C%5C%20a_%7B2%7D%5E%7B%282%29%7D%20%26%3D%26%20sigmoid%28%5Ctheta%20_%7B20%7D%5E%7B%281%29%7Dx_%7B0%7D%20&plus;%20%5Ctheta%20_%7B21%7D%5E%7B%281%29%7Dx_%7B1%7D%20&plus;%20%5Ctheta%20_%7B22%7D%5E%7B%281%29%7Dx_%7B2%7D%20&plus;%20%5Ctheta%20_%7B23%7D%5E%7B%281%29%7Dx_%7B3%7D%29%20%5C%5C%5C%5C%20a_%7B3%7D%5E%7B%282%29%7D%20%26%3D%26%20sigmoid%28%5Ctheta%20_%7B30%7D%5E%7B%281%29%7Dx_%7B0%7D%20&plus;%20%5Ctheta%20_%7B31%7D%5E%7B%281%29%7Dx_%7B1%7D%20&plus;%20%5Ctheta%20_%7B32%7D%5E%7B%281%29%7Dx_%7B2%7D%20&plus;%20%5Ctheta%20_%7B33%7D%5E%7B%281%29%7Dx_%7B3%7D%29%5C%5C%5C%5C%20h_%7B%5Ctheta%20%7D%28x%29%20%3D%20a_%7B1%7D%5E%7B%283%29%7D%20%26%3D%26%20sigmoid%28%5Ctheta%20_%7B10%7D%5E%7B%282%29%7Dx_%7B0%7D%20&plus;%20%5Ctheta%20_%7B11%7D%5E%7B%282%29%7Dx_%7B1%7D%20&plus;%20%5Ctheta%20_%7B12%7D%5E%7B%282%29%7Dx_%7B2%7D%20&plus;%20%5Ctheta%20_%7B13%7D%5E%7B%282%29%7Dx_%7B3%7D%29%20%5Cend%7Barray%7D)



因此这里![](http://latex.codecogs.com/gif.latex?%5Clarge%20%5Ctheta%20%5E%7B%281%29%7D%20%3D%20%5Cbegin%7Bbmatrix%7D%20%5Ctheta%20_%7B10%7D%20%26%20%5Ctheta%20_%7B11%7D%26%20%5Ctheta%20_%7B12%7D%26%20%5Ctheta%20_%7B13%7D%5C%5C%20%5Ctheta%20_%7B20%7D%26%20%5Ctheta%20_%7B21%7D%26%20%5Ctheta%20_%7B22%7D%26%20%5Ctheta%20_%7B23%7D%5C%5C%20%5Ctheta%20_%7B30%7D%20%26%20%5Ctheta%20_%7B31%7D%26%20%5Ctheta%20_%7B32%7D%26%20%5Ctheta%20_%7B33%7D%20%5Cend%7Bbmatrix%7D)

关于神经网络的最基本的知识就介绍到，下面举一些例子，帮助大家更好的理解神经网络。


**Example 1：实现“与”功能**       

![](http://img.blog.csdn.net/20160413215108293)

如上图所示，我们容易求得$h_{0}(x)=sigmoid(-30+20x_1+20x_2)$,因此我们可以画出$x_1,x_2$的真值表，看看它是怎样实现“与”功能的。

![](http://img.blog.csdn.net/20160413220626890)

因此，从上图可以看出是如何实现“与”功能的。。。

**Example 2:**    

实现“或”功能：

![](http://img.blog.csdn.net/20160413221821180)


**参考链接**           

[神经网络入门基础知识 neural networks basics](http://blog.csdn.net/u012328159/article/details/51143536)