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

**文字居中** 

```
text-align: center;
```


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








