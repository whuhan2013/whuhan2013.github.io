---
layout: post
title: 自定义View实例总结
date: 2016-6-20
categories: blog
tags: [自定义控件]
description: 自定义View实例总结
---

**这里的实例都是我看过源码实现之后的,并不是简单的摘抄** 

- [Android水波纹特效的简单实现 - 简书](http://www.jianshu.com/p/cba46422de67)

这种类型的水波纹，其实无非就是画圆而已，在给定的矩形中，一个个圆由最小半径扩大到最大半径，伴随着透明度从1.0变为0.0。我们假定这种扩散是匀速的，则一个圆从创建（透明度为1.0）到消失（透明度为0.0）的时长就是定值，那么某一时刻某一个圆的半径以及透明度完全可以由扩散时间（当前时间 - 创建时间）决定。

**效果如下**  

![](http://upload-images.jianshu.io/upload_images/2234662-e99193001cf9376b.gif?imageMogr2/auto-orient/strip)
