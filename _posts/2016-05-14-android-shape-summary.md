---
layout: post
title: Android之shape属性详解
date: 2016-5-14
categories: blog
tags: [android]
description: Android之shape属性详解
---   

 
有时候 ，为了满足一些需求，我们要用到 shape 去定义 一些背景，shape 的用法 跟图片一样 ，可以给View设置 Android:background=”@drawable/shape”, 定义的shape 文件，放在 res/shape 目录下

通常我们可以用shape 做 button 的背景选择器，也可以做切换tab 时，底部的下划线。

先看我们用shape 都可以做什么


shape下面 一共有6个子节点， 常用的 就只有 四个，padding 和size 一般用不到。 

- corners ———-圆角 
- gradient ———-渐变 
- padding ———-内容离边界距离 
- size　————大小　 
- solid 　———-填充颜色 
- stroke ———-描边


```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <!-- 圆角 -->
    <corners
        android:radius="9dp"
        android:topLeftRadius="2dp"
        android:topRightRadius="2dp"
        android:bottomLeftRadius="2dp"
        android:bottomRightRadius="2dp"/><!-- 设置圆角半径 -->
    <!-- 渐变 -->
    <gradient
        android:startColor="@android:color/white"
        android:centerColor="@android:color/black"
        android:endColor="@android:color/black"
        android:useLevel="true"
        android:angle="45"
        android:type="radial"
        android:centerX="0"
        android:centerY="0"
        android:gradientRadius="90"/>
    <!-- 间隔 -->
    <padding
        android:left="2dp"
        android:top="2dp"
        android:right="2dp"
        android:bottom="2dp"/><!-- 各方向的间隔 -->
    <!-- 大小 -->
    <size
        android:width="50dp"
        android:height="50dp"/><!-- 宽度和高度 -->

    <!-- 填充 -->
    <solid
        android:color="@android:color/white"/><!-- 填充的颜色 -->

    <!-- 描边 -->
    <stroke
        android:width="2dp"
        android:color="@android:color/black"
        android:dashWidth="1dp"
        android:dashGap="2dp"/>
</shape>
```


### shape 做虚线

拿shape 做虚线,shape 设置为line ， stroke 是描边属性， 
其中 dashGap dashWidth 两个属性彼此一起存在才生效。

dashGap ：两段之间的空隙宽度、 
dashWidth ：一段线的宽度


```
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line" >
    <stroke
        android:dashGap="3dp"
        android:dashWidth="8dp"
        android:width="1dp"
        android:color="#009999" />
</shape>
```

### 效果如下 

![](http://img.blog.csdn.net/20150708154456297)


### shape做渐变实线

gradient 表示渐变             
angle 渐变角度，45的倍数。                                    
startColor endColor centerColor 起 止 中 的颜色          


```
<shape xmlns:android="http://schemas.android.com/apk/res/android" >

    <gradient
        android:type="linear"
        android:angle="0"
        android:endColor="#F028A2"
        android:startColor="#2A99F3" />
</shape>
```

### 效果如下 

![](http://img.blog.csdn.net/20150708154620189)


### shape 做view背景选择器

这里注意 ，item 的 state_pressed=true 是选择状态，按下，另一个不设置 就是 正常状态。                       
solid ：是填充颜色                       
corners：设置 四个角的弧度                    


```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"  >
        <shape xmlns:android="http://schemas.android.com/apk/res/android"
            android:shape="rectangle">

            <!--填充色  -->
            <solid 
                android:color="#ffffff" />  

            <!-- 描边 -->  
            <!-- dashGap：- 与- 虚线之间的间隔  dashWidth: 实线的宽度width： color:-->
         <!--       <stroke              
                android:dashGap="10dp"
                android:dashWidth="5dp"
                android:width="1dip"
                android:color="#d3d3d3"
                /> -->
            <!-- 圆角 -->  
            <corners
                android:topRightRadius="10dp"     
                android:bottomLeftRadius="10dp"   
                android:topLeftRadius="10dp"    
                android:bottomRightRadius="10dp"    
                />
        </shape>
    </item>
    <item >
        <!--shape:oval 椭圆     rectangle:方形    line：线性-->
        <shape xmlns:android="http://schemas.android.com/apk/res/android"

            android:shape="rectangle">
            <gradient  
                android:startColor="#55B4FE"  
                android:endColor="#3d8FFB" 
                android:type="linear"/>
            <!-- 描边 -->  
         <!--       <stroke 
               android:dashGap="10dp"
                android:dashWidth="5dp"
                android:width="1dip"
                android:color="#d3d3d3"
                /> -->
            <!-- 圆角  上下左右四个角 弧度-->  
            <corners
                android:topRightRadius="10dp"     
                android:bottomLeftRadius="10dp"   
                android:topLeftRadius="10dp"    
                android:bottomRightRadius="10dp"    
                />   
        </shape>
    </item>

</selector>
```

### 效果如下

![](http://img.blog.csdn.net/20150708155208695)


### shape 做矩形

android:shape=”rectangle”选为矩形

```
<?xml version="1.0" encoding="utf-8"?>
<shape  xmlns:android="http://schemas.android.com/apk/res/android" 
    android:shape="rectangle">



    <!-- 渐变 -->
    <gradient
        android:type="linear"
        android:startColor="@android:color/holo_blue_bright"
        android:centerColor="@android:color/holo_green_dark"
        android:endColor="@android:color/holo_red_light"
        android:useLevel="true"
        android:angle="45"/>




    <!-- 填充 -->
<!--     <solid
        android:color="@android:color/white"/>填充的颜色 -->

    <!-- 描边 -->
    <stroke
        android:width="2dp"
        android:color="@android:color/black"/>

</shape>
```

### 效果如下  

![](http://img.blog.csdn.net/20150708155824480)


### shape 作描边矩形 和 椭圆


shape 作描边矩形 和 椭圆         
这里注意shape                            
android:shape=”oval” 椭圆             


```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" 
    android:shape="rectangle">


    <!-- 填充 -->
    <solid
        android:color="@android:color/holo_blue_bright"/><!-- 填充的颜色 -->

    <!-- 描边 -->
    <stroke
        android:width="2dp"
        android:color="@android:color/black"
        android:dashWidth="1dp"
        android:dashGap="2dp"/>

    <corners 
      android:topLeftRadius="20dp"            
android:topRightRadius="20dp"           
android:bottomLeftRadius="20dp"     
android:bottomRightRadius="20dp" 
        android:radius="50dp"/>
</shape>
```

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" 
    android:shape="oval">


    <!-- 填充 -->
    <solid

        android:color="@android:color/holo_orange_light"/><!-- 填充的颜色 -->

    <!-- 描边 -->
    <stroke
        android:width="2dp"
        android:color="@android:color/black"/>

</shape>
```

### 效果如下  

![](http://img.blog.csdn.net/20150708160002191)

### 代码

[ShapeDemo代码](http://download.csdn.net/detail/u011733020/8880615)

### 参考链接

[*Android shape属性整理 - Because if I don’t write it down, I’ll forget it - 博客频道 - CSDN.NET](http://blog.csdn.net/u011733020/article/details/46804817)


