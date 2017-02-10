---
layout: post
title: 机器学习可行性分析
date: 2017-2-10
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

机器学习的可行性分析。

一， 第一条准则： 没有免费的午餐！(no free lunch !)              
给一堆数据D， 如果任何未知的f （即建立在数据D上的规则）都是有可能的，那么从这里做出有意义的推理是不可能的！！ doomed !!          
如下面这个问题无解（或者勉强说没有唯一解）           
![](https://img3.doubanio.com/view/note/large/public/p10434643.jpg)            
下面这题也是如此：        
![](https://img3.doubanio.com/view/note/large/public/p10434664.jpg)        
如何解决上述存在的问题？ 答：做出合理的假设。        

#### 关于罐子里选小球的推论       
霍夫丁不等式(Hoeffding’s Inequality)         
![](https://img5.doubanio.com/view/note/large/public/p10435006.jpg)
这里v 是样本概率；u 是总体概率        

#### 罐子理论与学习问题的联系      

![](https://img1.doubanio.com/view/note/large/public/p10435037.jpg)      
对于一个固定的假设h， 我们需要验证它的错误率；然后根据验证的结果选择最好的h。
![](https://img3.doubanio.com/view/note/large/public/p10435094.jpg)     



