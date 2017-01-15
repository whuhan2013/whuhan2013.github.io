---
layout: post
title: 机器学习实例Photo OCR
date: 2017-1-15
categories: blog
tags: [机器学习]
description: 机器学习
---


**问题描述和流程图**      
图像文字识别应用所作的事是,从一张给定的图片中识别文字。这比从一份扫描文档中识别文字要复杂的多。           
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class13/p1.png)
 为了完成这样的工作,需要采取如下步骤:                             
1. 文字侦测(Text detection)——将图片上的文字与其他环境对象分离开来       
2. 字符切分(Character segmentation)——将文字分割成一个个单一的字符       
3. 字符分类(Character classification)——确定每一个字符是什么       

可以用任务流程图来表达这个问题,每一项任务可以由一个单独的小队来负责解决:
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class13/p2.png)

**滑动窗口**       
滑动窗口是一项用来从图像中抽取对象的技术。假使我们需要在一张图片中识别行人, 首先要做的是用许多固定尺寸的图片来训练一个能够准确识别行人的模型。然后我们用之前 训练识别行人的模型时所采用的图片尺寸在我们要进行行 人识别的图片上进行剪裁,然后 将剪裁得到的切片交给模型,让模型判断是否为行人,然后在图片上滑动剪裁区域重新进行 剪裁,将新剪裁的切片也交给模型进行判断,如此循环直至将图片全部检测完。
一旦完成后,我们按比例放大剪裁的区域,再以新的尺寸对图片进行剪裁,将新剪裁的 切片按比例缩小至模型所采纳的尺寸,交给模型进行判断,如此循环
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class13/p3.png)   
滑动窗口技术也被用于文字识别,首先训练模型能够区分字符与非字符,然后,运用滑动窗口技术识别字符,一旦完成了字符的识别,我们将识别得出的区域进行一些扩展,然后 将重叠的区域进行合并。接着我们以宽高比作为过滤条件,过滤掉高度比宽度更大的区域 (认为单词的长度通常比高度要大)。下图中绿色的区域是经过这些步骤后被认为是文字的 区域,而红色的区域是被忽略的。       

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class13/p4.png) 
以上便是字符切分阶段。 最后一个阶段是字符分类阶段,利用神经网络、支持向量机 或者逻辑回归算法训练一个分类器即可。   