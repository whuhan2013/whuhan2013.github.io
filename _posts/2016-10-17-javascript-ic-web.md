---
layout: post
title: 企业网站实例
date: 2016-10-17
categories: blog
tags: [javascript]
description: 企业网站实例
---

**本文主要包括复习html+css相关内容**  

**浮动**          
浮动分为左浮动与右浮动，浮动后飘浮在常用元素上，不占位置，所以需要清除浮动。      
同时浮动时不会撑开父元素，使得父元素没有高度与宽度.     

1、给父级DIV也加上float；

2、给父级DIV加上height；

3、在子级DIV后面加上<div style="clear:both;"></div>，即清除浮动。

上面的方法中，可能第三种用的多一点（至少我是这样的），昨天发现原来还有一个更简单的方法，就是给父元素加上overflow:hidden;属性，代码如下：

```
<div style="overflow:hidden;">
    <div style="float:left;">test</div>
</div>
```

**元素居中方法**  

水平居中

```
margin: 0 auto;
```

垂直居中：将line-height设为与父元素一致即可.

**图片与旁边的文字中间对齐**   

```
.logo_right img{
  vertical-align: middle;
}
```

水平居中规定width之后用margin:0 auto;就可以了。另外关于居中的方法很多。这里再告诉你一种：

```
.Absolute-Center { 
margin: auto; 
position: absolute; 
top: 0; left: 0; bottom: 0; right: 0; 
} 
```

这个用在规定过高度的元素上可以水平且垂直居中。算是我看过兼容性各项比较好的。ie8以上都可以。

**文字居中** 

```
text-align: center;
```


**display属性**         

- none  此元素不会被显示。
- block 此元素将显示为块级元素，此元素前后会带有换行符。
- inline  默认。此元素会被显示为内联元素，元素前后没有换行符。
- inline-block  行内块元素。（CSS2.1 新增的值）
- table 此元素会作为块级表格来显示（类似 <table>），表格前后带有换行符。    

**position**       

absolute            
生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。
元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。

fixed            
生成绝对定位的元素，相对于浏览器窗口进行定位。
元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。   

relative               
生成相对定位的元素，相对于其正常位置进行定位。
因此，"left:20" 会向元素的 LEFT 位置添加 20 像素。

static         
默认值。没有定位，元素出现在正常的流中（忽略 top, bottom, left, right 或者 z-index 声明）。

inherit           
规定应该从父元素继承 position 属性的值。


**myFocus库实现焦点图**  

```
<script type="text/javascript" src="js/myfocus-2.0.1.min.js" charset="utf-8"></script><!--引入myFocus库-->

  <script type="text/javascript">
    myFocus.set({
      id:'boxID',//焦点图盒子ID
      pattern:'mF_fancy',//风格应用的名称
      time:3,//切换时间间隔(秒)
      trigger:'click',//触发切换模式:'click'(点击)/'mouseover'(悬停)
      width:1000,//设置图片区域宽度(像素)
      height:310,//设置图片区域高度(像素)
      txtHeight:'default'//文字层高度设置(像素),'default'为默认高度，0为隐藏
});
  </script>

  <div class="ad">
    <div id="boxID"><!--焦点图盒子-->
      <div class="loading"><img src="images/loading.gif" alt="请稍候..." /></div>
      <!--载入画面(可删除)-->
      <div class="pic"><!--内容列表(li数目可随意增减)-->
        <ul>
          <li><a href="#"><img src="images/ad2.jpg" thumb="" alt="" text="详细描述2" /></a></li>
          <li><a href="#"><img src="images/ad3.jpg" thumb="" alt="" text="详细描述3" /></a></li>
          <li><a href="#"><img src="images/ad4.jpg" thumb="" alt="" text="详细描述4" /></a></li>
          <li><a href="#"><img src="images/ad3.jpg" thumb="" alt="" text="详细描述5" /></a></li>
        </ul>
      </div>
    </div>
  </div>

```


**新闻中心制作**  

**源码：** [新闻中心](https://github.com/whuhan2013/freeCodeCampProject/blob/master/newsCenter.html)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p2.png)

**嵌套列表制作** 

```
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>无标题文档</title>
</head>

<body>
<!--在此制作一个嵌套列表-->
<div class="mybody">
  <ul>
    <li>首页</li>
    <li>课程中心
      <ul>
        <li>Web前端
          <ul>
            <li>HTML</li>
            <li>CSS</li>
            <li>JavaScript</li>
            <li>JQuery</li>
          </ul>
        </li>
        <li>Android开发</li>
        <li>PHP开发</li>
      </ul>
    </li>

  </ul>
</div>
</body>
</html>
````

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p3.png)


**企业网站完整项目**  

分为  

- 首页
- 新闻列表页
- 新闻详情页

**源代码:**[ICNWeb](https://github.com/whuhan2013/freeCodeCampProject/tree/master/INCWeb)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p4.png)

**编程挑战**

小伙伴们，综合所学知识，制作如下所示的页面布局。

要求：页面宽度1000px，水平居中。


**源代码：**[MuKe挑战](https://github.com/whuhan2013/freeCodeCampProject/tree/master/MUKEparactice)

**效果图**   

![](http://img.mukewang.com/53edabd70001532811161020.jpg)





