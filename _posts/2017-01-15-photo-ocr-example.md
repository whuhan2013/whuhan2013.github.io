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


