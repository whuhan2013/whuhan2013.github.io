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

**获取大量数据和人工数据**         
如果我们的模型是低方差的,那么获得更多的数据用于训练模型,是能够有更好的效果的。问题在于,我们怎样获得数据,数据不总是可以直接获得的,我们有可能需要人工地创 造一些数据。以我们的文字识别应用为例,我们可以字体网站下载各种字体,然后利用这些不同的字 体配上各种不同的随机背景图片创造出一些用于训练的实例,这让我们能够获得一个无限大 的训练集。这是从零开始创造实例。
另一种方法是,利用已有的数据,然后对其进行修改,例如将已有的字符图片进行一些扭曲、旋转、模糊处理。只要我们认为实际数据有可能和经过这样处理后的数据类似,我们 便可以用这样的方法来创造大量的数据。         
有关获得更多数据的几种方法:        
1. 人工数据合成        
2. 手动收集、标记数据        
3. 众包        

**上限分析**      
在机器学习的应用中,我们通常需要通过几个步骤才能进行最终的预测,我们如何能够知道哪一部分最值得我们花时间和精力去改善呢?这个问题可以通过上限分析来回答。         
 回到我们的文字识别应用中,我们的流程图如下:      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class13/p2.png)
流程图中每一部分的输出都是下一部分的输入,上限分析中,我们选取一部分,手工  供 100%正确的输出结果,然后看应用的整体效果 升了多少。假使我们的例子中总体效果 为 72%的正确率。
如果我们令文字侦测部分输出的结果 100%正确,发现系统的总体效果从 72% 高到了 89%。这意味着我们很可能会希望投入时间精力来 高我们的文字侦测部分。
接着我们手动选择数据,让字符切分输出的结果 100%正确,发现系统的总体效果只  升了 1%,这意味着,我们的字符切分部分可能已经足够好了。
最后我们手工选择数据,让字符分类输出的结果 100%正确,系统的总体效果又 升了 10%,这意味着我们可能也会应该投入更多的时间和精力来 高应用的总体表现。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class13/p5.png) 

