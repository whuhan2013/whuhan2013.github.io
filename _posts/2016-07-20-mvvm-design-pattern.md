---
layout: post
title: MVVM架构学习
date: 2016-7-20
categories: blog
tags: [设计模式]
description: 设计模式
---


典型的MVC应用里，许多逻辑被放在View Controller中，他们中一些确实属于View Controller，但更多的是表现逻辑，即将Model中数据转换为View可以呈现的内容的事情。例如将JSON包里的某个NSDate转换为特定格式的NSString。这也导致了MVC被人称作Massive-View-Controller（重量级视图控制器）。

通常Controller中应该只放如下代码：

- 初始化时构造相应的View和Model
- 监听Model层的事件，将Model层的数据传递到View层
- 监听View层的时间，将View层的事件传递到Model层

仅此而已，除此之外的任何逻辑都不应该放到Controller中。因此这也就有了MVVM

**MVVM**            
下图所示为MVVM设置：MVVM其实就是MVC的增强版。我们正式连接了View 和View Controller，并将表示逻辑从Controller中移出，放到了一个新的对象里，即View Model中。

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/designpattern/p2.png)


这样做可带来如下的益处：

- 减少View Controller的复杂性，使得表示逻辑易于测试。
- 兼容MVC模式
- MVVM 配合一个绑定机制效果最好


