---
layout: post
title: 机器学习基石之过拟合
date: 2017-2-19
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

**1，什么是过拟合(overfitting)**                  
简单的说就是这样一种学习现象：Ein 很小，Eout 却很大。        
而Ein 和 Eout 都很大的情况叫做 underfitting。           
这是机器学习中两种常见的问题。         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p1.jpg)      

上图中，竖直的虚线左侧是"underfitting", 左侧是"overfitting”。

发生overfitting 的主要原因是：（1）使用过于复杂的模型(dvc 很大)；（2）数据噪音；（3）有限的训练数据。

**2，噪音与数据规模**        
我们可以理解地简单些：有噪音时，更复杂的模型会尽量去覆盖噪音点，即对数据过拟合！         
这样，即使训练误差Ein 很小（接近于零），由于没有描绘真实的数据趋势，Eout 反而会更大。         
即噪音严重误导了我们的假设。           

还有一种情况，如果数据是由我们不知道的某个非常非常复杂的模型产生的，实际上有限的数据很难去“代表”这个复杂模型曲线。我们采用不恰当的假设去尽量拟合这些数据，效果一样会很差，因为部分数据对于我们不恰当的复杂假设就像是“噪音”，误导我们进行过拟合。

如下面的例子，假设数据是由50次幂的曲线产生的（下图右边），与其通过10次幂的假设曲线去拟合它们，还不如采用简单的2次幂曲线来描绘它的趋势。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p2.jpg)    

**3，随机噪音与确定性噪音 (Deterministic Noise)**        
之前说的噪音一般指随机噪音(stochastic noise)，服从高斯分布；还有另一种“噪音”，就是前面提到的由未知的复杂函数f(X) 产生的数据，对于我们的假设也是噪音，这种是确定性噪音。       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p3.jpg)    
上图是关于2次曲线和10次曲线对数据的拟合情况，我们将overfit measure 表示为Eout(g10) - Eout(g2)。

下图左右两边分别表示了随机噪音和确定性噪音对于Overfitting 的影响。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter12/p4.jpg) 

可见，数据规模一定时，随机噪音越大，或者确定性噪音越大（即目标函数越复杂），越容易发生overfitting。总之，容易导致overfitting 的因素是：数据过少；随机噪音过多；确定性噪音过多；假设过于复杂(excessive power)。

如果我们的假设空间不包含真正的目标函数f(X)（未知的），那么无论如何H 无法描述f(X) 的全部特征。这时就会发生确定性噪音。它与随机噪音是不同的。
我们可以类比的理解它：在计算机中随机数实际上是“伪随机数”，是通过某个复杂的伪随机数算法产生的，因为它对于一般的程序都是杂乱无章的，我们可以把伪随机数当做随机数来使用。确定性噪音的哲学思想与之类似。:-)


**4，解决过拟合问题**            
对应导致过拟合发生的几种条件，我们可以想办法来避免过拟合。              
(1) 假设过于复杂(excessive dvc) => start from simple model           
(2) 随机噪音  => 数据清洗                                 
(3) 数据规模太小 => 收集更多数据，或根据某种规律“伪造”更多数据               
正规化(regularization) 也是限制模型复杂度的，在下一讲介绍。                

4.1 数据清洗(data ckeaning/Pruning)               
将错误的label 纠正或者删除错误的数据。                    

4.2 Data Hinting: “伪造”更多数据, add "virtual examples"           
例如，在数字识别的学习中，将已有的数字通过平移、旋转等，变换出更多的数据         

