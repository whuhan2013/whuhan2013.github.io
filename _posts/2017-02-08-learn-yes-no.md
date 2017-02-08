---
layout: post
title: 机器学习基石之二元分类
date: 2017-2-8
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

**二元分类问题**         

x = (x1, x2, ..., xd)   ----  features        
w = (w1, w2, ..., wd) ----  未知（待求解）的权重       

对于银行是否发送信用卡问题：        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p1.jpg)
perceptron 假设： 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p2.jpg)
sign 是取符号函数， sign(x) = 1 if x>0, -1 otherwise           
向量表示：   
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p3.jpg)
感知机（perceptron）是一个线性分类器(linear classifiers）。           
线性分类器的几何表示：直线、平面、超平面     

#### Perceptron Learning Algorithm (PLA)

感知机求解（假设空间为无穷多个感知机；注意区分下面的普通乘法和向量内积，内积是省略了向量转置的表示）        
初始w = 0 （零向量）。             
第一步：找一个分错的数据（xi, yi）， sign(w*xi) != yi；        
第二步：调整w 的偏差，w = w + yi*xi；                    
循环第一、二步，直到没有任何分类错误， 返回最后得到的w。        

实际操作时，寻找下一个错误数据可以按照简单的循环顺序进行（x1, x2, ..., xn）；如果遍历了所有数据没有找到任何一个错误，则算法终止。注：也可以预先计算（如随机）一个数据序列作为循环顺序。

以上为最简单的PLA 算法。没有解决的一个基本问题是：该算法不一定能停止

**PLA 算法是否能正常终止**         
分两种情况讨论：数据线性可分；数据线性不可分。       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p4.jpg)
注意PLA 停止的条件是，对任何数据分类都正确，显然数据线性不可分时PLA 无法停止，这个稍后研究。

1， 我们先讨论线性可分的情况。          
数据线性可分时，一定存在完美的w(记为wf)， 使得所有的(xi, yi), yi = sign(wf*xi).
可知：    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p5.jpg)
下面证明在数据线性可分时，简单的感知机算法会收敛。      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p6.jpg)
而且量向量夹角余弦值不会大于1，可知T 的值有限。由此，我们证明了简单的PLA 算法可以收敛。

**数据线性不可分：Pocket Algorithm**        
当数据线性不可分时（存在噪音），简单的PLA 算法显然无法收敛。我们要讨论的是如何得到近似的结果。      
我们希望尽可能将所有结果做对，即        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p7.jpg)
寻找wg 是一个NP-hard 问题！
只能找到近似解。

Pocket Algorithm
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter2/p8.jpg)
与简单PLA 的区别：迭代有限次数（提前设定）；随机地寻找分错的数据（而不是循环遍历）；只有当新得到的w 比之前得到的最好的wg 还要好时，才更新wg（这里的好指的是分出来的错误更少）。
由于计算w 后要和之前的wg 比较错误率来决定是否更新wg， 所以pocket algorithm 比简单的PLA 方法要低效。

最后我们可以得到还不错的结果：wg。

