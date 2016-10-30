---
layout: post
title: Bootstrap学习
date: 2016-10-30
categories: blog
tags: [javascript]
description: Bootstrap学习
---

Bootstrap框架属于UI框架，这个和jQuery不太一样，其实准确的描述Bootstrap框架属于css框架而非javascript框架，但是它本身也使用javascript来完善Bootstrap框架的视觉效果。此外，Bootstrap框架十分超前，在支持html5和css3的浏览器上表现特别好，而且对移动终端的浏览器支持也是相当优秀。

**一个完整的Bootstrap框架包含如下四个部分：**

- 脚手架（不知道官网为啥这么翻译）：用于重置背景、链接样式、栅格系统等，并包含两个简单的布局结构。Bootstrap的样式使用了lesscss技术，比如重置背景这样的操作，这些比较简单我就不展开叙述了，脚手架里最出彩的是栅格系统和布局。栅格系统是将页面宽度分成12列，栅格系统分为两种类型，一种是默认栅格系统，这时候栅格系统是按940px像素进行等分，我们可以使用span1，span4这样的class属性操作默认栅格布局，另一种是流式栅格系统，这个时候分列的宽度就不是固定，而是根据你可视页面进行12等分，同样可以使用span1，span4操作流式栅格。这个系统非常之好，做css最难的就是div布局，使用栅格系统可以大大简化div的布局操作。另外一个就是做布局操作了，布局也分为固定和流式，让不太精通css布局也能轻松操作布局。
- 基本的css样式。Bootstrap给出了一样常用的HTML元素的样式，例如：按钮、表单和文字等等。大部分做网站的人都不是美工出身，做出赏心悦目的网页是件很困难的事情，css提供的样式很专业很精美，能让我们轻松开发出一套精美的网站
- Css组件：Bootstrap还提供一些常用的css组件，同样很优秀很棒。
- Javascript插件：Bootstrap是个开放的系统，我们可以随意扩展Bootstrap，特别是javascript的框架，这样Bootstrap就会更加专业。


**引入boostrap**     

```
<!-- 新 Bootstrap 核心 CSS 文件 -->
<link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap.min.css">

<!-- 可选的Bootstrap主题文件（一般不用引入） -->
<link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap-theme.min.css">

<!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
<script src="http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>

<!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
<script src="http://cdn.bootcss.com/bootstrap/3.3.0/js/bootstrap.min.js"></script>
```

#### 布局容器
Bootstrap 需要为页面内容和栅格系统包裹一个 .container 容器。我们提供了两个作此用处的类。注意，由于 padding 等属性的原因，这两种 容器类不能互相嵌套。

- .container 类用于固定宽度并支持响应式布局的容器。       
- .container-fluid 类用于 100% 宽度，占据全部视口（viewport）的容器。


#### 栅格系统
Bootstrap 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多12列。

![](http://images.cnitblog.com/blog/270324/201402/211703166921878.png)

#### 模板文件        
包括按钮，表格，标签等的样式,代码参见：[template.html](https://github.com/whuhan2013/freeCodeCampProject/blob/master/bootstrap/template.html)

可以通过这里详细学习：[玩转Bootstrap（基础）](http://www.imooc.com/learn/141)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p14.png)




