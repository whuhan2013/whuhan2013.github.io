---
layout: post
title: 颜色定位与偏斜扭转
date: 2017-1-16
categories: blog
tags: [Opencv]
description: 颜色定位与偏斜扭转
---

在前面的介绍里，我们使用了Sobel查找垂直边缘的方法，成功定位了许多车牌。但是，Sobel法最大的问题就在于面对垂直边缘交错的情况下，无法准确地定位车牌。例如下图。为了解决这个问题，可以考虑使用颜色信息进行定位。              
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p14.jpg)
如果将颜色定位与Sobel定位加以结合的话，可以使车牌的定位准确率从75%上升到94%。     


