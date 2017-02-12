---
layout: post
title: 机器学习基石之归纳理论
date: 2017-2-12
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

#### 界函数（bounding function）的概念                
是指当（最小）突破点为k 时，生长函数m(N) 可能的最大值，记为B(N, k)。                              
显然，当k=1时，B(N, 1) = 1; 当k > N 时，B(N,k) = 2^N; 当k = N 时，B(N,k)=2^N - 1.                    
于是很容易得到Bounding function table:            
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter6/p1.jpg)

