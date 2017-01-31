---
layout: post
title: AdaBoost算法学习
date: 2017-1-31
categories: blog
tags: [机器学习]
description: 机器学习
---

在介绍AdaBoost算法之前，需要了解一个类似的算法，装袋算法(bagging)，bagging是一种提高分类准确率的算法，通过给定组合投票的方式，获得最优解。比如你生病了，去n个医院看了n个医生，每个医生给你开了药方，最后的结果中，哪个药方的出现的次数多，那就说明这个药方就越有可能性是最优解，这个很好理解。而bagging算法就是这个思想。         

而AdaBoost算法的核心思想还是基于bagging算法，但是他又一点点的改进，上面的每个医生的投票结果都是一样的，说明地位平等，如果在这里加上一个权重，大城市的医生权重高点，小县城的医生权重低，这样通过最终计算权重和的方式，会更加的合理，这就是AdaBoost算法。AdaBoost算法是一种迭代算法，只有最终分类误差率小于阈值算法才能停止，针对同一训练集数据训练不同的分类器，我们称弱分类器，最后按照权重和的形式组合起来，构成一个组合分类器，就是一个强分类器了。算法的只要过程：            
1、对D训练集数据训练处一个分类器Ci                
2、通过分类器Ci对数据进行分类，计算此时误差率                        
3、把上步骤中的分错的数据的权重提高，分对的权重降低，以此凸显了分错的数据。         

算法流程如下：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter7/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter7/p2.png)

