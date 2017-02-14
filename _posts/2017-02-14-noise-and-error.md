---
layout: post
title: 机器学习基石之噪音和错误
date: 2017-2-14
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

当我们面对的问题不是完美的（无噪音）二值分类问题，VC 理论还有效吗？

1，噪音和非确定性目标

几种错误：(1) noise in y: mislabeled data; (2) noise in y: different labels for same x; (3) noise in x: error x.    
将包含噪音的y 看作是概率分布的，y ~ P(y|x)。          
学习的目标变为预测最有可能出现的y，max {P(y|x)}。             
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p2.png)

**2， 错误的测量 (error measure)**             
对错误不同的测量方法将对学习过程有不同的指导。          
如下面的0/1 error  和 squared error.           
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p3.jpg)
我们在学习之前，需要告诉模型我们所关心的指标或衡量错误的方法。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p4.png)

**3，衡量方法的选择**        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p5.jpg)

图中的false accept 和 false reject 就是我们在分类中常说的 false positive 和 false negative，只是台湾和大陆的叫法不同而已。   
采用0/1 error 时，对于每种错误，都会有相同程度的惩罚(panalize)。    

然而，在实际应用中，两种错误带来的影响可能很不一样。                 
例如（讲义中的例子），一个商场对顾客进行分类，老顾客可以拿到折扣，新顾客没有折扣；这时，如果发生false reject（将老顾客错分为新顾客）而不给其打折，那么会严重影响用户体验，对商家口碑造成损失，影响较坏；而如果发生false accept （将新顾客错分为老顾客)而给其打折，也没什么大不了的。          

另一个例子，CIA 的安全系统身份验证，如果发生false accept（将入侵者错分为合法雇员），后果非常严重。         

通过上述两个例子，我们知道，对于不同application，两类错误的影响是完全不同的，因此在学习时应该通过赋予不同的权重来区分二者。比如对于第二个例子，当发生false accept : f(x)=-1, g(x)=+1, 对Ein 加一个很大的值。            
Just remember: error is application/user-dependent         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter8/p6.png)

