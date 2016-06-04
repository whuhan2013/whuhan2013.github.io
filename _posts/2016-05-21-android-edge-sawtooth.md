---
layout: post
title: Android实现边缘凹凸的View
date: 2016-5-21
categories: blog
tags: [自定义控件]
description: Android实现边缘凹凸的View
---   

### 转载

最近做项目的时候遇到一个卡劵的效果，由于自己觉得用图片来做的话可以会出现适配效果不好，再加上自己自定义view方面的知识比较薄弱，所以想试试用自定义View来实现。但是由于自己知识点薄弱，一开始居然想着用画矩形来设置边缘实现，后面一个哥们指导了我，在这里感谢他。 


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaB5a5ic5ia0GKloj1rpcjumkx8E6YQE2anQGum2Avx0jH3ulmghCbIib5OQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

#### 实现分析


上面的图片其实和普通的Linearlayout，RelativeLayout一样，只是上下两边多了类似于半圆锯齿的形状。那么只需要处理不同地方。可以在上下两条线上画一个个白色的小圆来实现这种效果。

假如我们上下线的半圆以及半圆与半圆之间的间距是固定的，那么不同尺寸的屏幕肯定会画出不同数量的半圆，那么我们只需要根据控件的宽度来获取能画的半圆数。

大家观察图片，很容易发现，圆的数量总是圆间距数量－1，也就是，假设圆的数量是circleNum,那么圆间距就是circleNum+1。

所以我们可以根据这个计算出circleNum. 

circleNum = (int) ((w-gap)/(2*radius+gap)); 

这里gap就是圆间距，radius是圆半径，w是view的宽。 


### 具体的实现

下面看代码：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBRHCUw9lvJLwKTdRSoFd5qhpvGZNOm9gL8oWGefbJW1GqjjUy15WFgA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

上面定义了圆的半径和圆间距，同时初始化了这些值并且获取了需要画的圆数量。

接下来只需要一个一个将圆画出来就可以了。


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBaXATdwqCfORSaeG471LVOicWia5gXYRrM75yU24uCU5FCZLwla5k6ATg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


简单的根据circleNum的数量进行了圆的绘制。

这里remain/2是因为，可以一些情况，计算出来的可以画的数量不是刚好整除的。这样就会出现右边最后一个间距会比其它的间距都要宽。

所以我们在绘制第一个的时候加上了余下的间距的一半，即使是不整除的情况。至少也能保证第一个和最后一个间距宽度一致。

这样就实现了，看看布局文件：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBvNiaS6ibnafgAP1xFZ8BzMlhkqpAOnWWJU3w6a2gib5fVdIQic7CvSPpcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

效果图：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBSETbqSWJiaahE2UibicBibxJudqa45OjRZz5xRtt4vWf1qqgu8H7j1ibLKA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

作者通过非常巧妙和简单的方式实现了边缘特殊图像的绘制，相信大家可以轻松的学习到相关知识，不过这里指定了特定的颜色（白色），如果想要直接像橡皮擦一样擦成透明，可以使用xfermode来做处理。


### 源代码下载
[源代码](http://download.csdn.net/detail/yissan/9524401)

### 参考链接

[一起来学习android自定义控件—边缘凹凸的View](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820217&idx=1&sn=69380a847716dc4c3caca4a702df6f0d&scene=0#wechat_redirect)凹凸


### 更新

**利用边缘凹凸的View来实现比较好看的editText**

**效果如下**

![这里写图片描述](http://img.blog.csdn.net/20160604213217847)


### 布局文件 

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="0dp"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:background="@color/backcircle"
    tools:context="com.zj.second.MainActivity">


    <com.zj.second.CouponDisplayView
        android:layout_marginRight="16dp"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#FFFFFF"
        android:padding="0dp">

        <EditText
            android:padding="20dp"
            android:textStyle="bold"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="姓 名"
            android:background="@null"
            android:textSize="16sp"/>

        <View
            android:layout_width="fill_parent"
            android:layout_height="1dip"
            android:background="#FF909090" />

        <EditText
            android:padding="20dp"
            android:textStyle="bold"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="电 话"
            android:background="@null"
            android:textSize="16sp"/>

        <View
            android:layout_width="fill_parent"
            android:layout_height="1dip"
            android:background="#FF909090" />

        <EditText
            android:padding="20dp"
            android:textStyle="bold"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="地 址"
            android:background="@null"
            android:textSize="16sp"/>

        <View
            android:layout_width="fill_parent"
            android:layout_height="1dip"
            android:background="#FF909090" />

    <EditText

        android:gravity="top|left"
        android:padding="20dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="案情描述"
        android:minLines="8"
        android:background="@null"

        />
    </com.zj.second.CouponDisplayView>

    <ImageView

        android:id="@+id/imageView_send"
        android:layout_width="36dp"
        android:layout_height="36dp"
        android:padding="0dp"
        android:layout_marginLeft="3dp"
        android:layout_marginTop="168dp"
        android:foreground="?android:attr/selectableItemBackground"
        android:src="@drawable/inline_reply_dialog_send"

        android:layout_alignParentRight="true"

        />

</RelativeLayout>
```



### 参考链接

[文本框 - Material Design 中文版 - 极客学院Wiki](http://wiki.jikexueyuan.com/project/material-design/components/text-fields.html) 




