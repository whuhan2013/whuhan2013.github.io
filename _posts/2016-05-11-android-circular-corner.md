---
layout: post
title: Android之圆角矩形
date: 2016-5-11
categories: blog
tags: [android]
description: Android之圆角矩形
---   

### 安卓圆角矩形的定义

在drawable文件夹下，定义corner.xml  

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" 
android:shape="rectangle">
<!-- 填充颜色 -->
<solid android:color="#FF8C69"></solid>

<!-- 线的宽度，颜色灰色 -->
<stroke android:width="1dp" android:color="#D5D5D5"></stroke> 

<!-- 矩形的圆角半径 -->
<corners android:radius="10dp" /> 

</shape>
```

### 具体使用  

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="${relativePackage}.${activityClass}" >
    
    <ImageView 
        android:adjustViewBounds="true"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:background="@drawable/b"
        android:padding="0dp"
        
        />
<LinearLayout 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal"
    android:layout_weight="1"
    >
    <LinearLayout
        android:id="@+id/l1"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_margin="3dp"
        android:layout_marginBottom="5dp"
        android:layout_weight="1"
        android:background="@drawable/corner"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:text="二手市场" />
    </LinearLayout>

    <LinearLayout
        android:id="@+id/l2"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_margin="3dp"
        android:layout_weight="1"
        android:background="@drawable/corner2"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="失物招领" />
    </LinearLayout>
</LinearLayout>

<LinearLayout 
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal"
    android:layout_weight="1"
    >
    <LinearLayout 
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:orientation="vertical"
        
        >
        <LinearLayout
        android:id="@+id/l3"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
   
        android:layout_margin="3dp"
        android:layout_weight="1"
        android:background="@drawable/corner3"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="寻物启事" />
    </LinearLayout>
    <LinearLayout
        
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_margin="3dp"
        android:layout_weight="1"
        android:background="@drawable/corner4"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="表白墙" />
    </LinearLayout>
    </LinearLayout>
    <LinearLayout 
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_weight="1"
        android:orientation="horizontal"
        >
        <LinearLayout
        
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_margin="3dp"
        android:layout_weight="1"
        android:background="@drawable/corner5"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="生活查询" />
    </LinearLayout>
    <LinearLayout
        
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:layout_margin="3dp"
        android:layout_weight="1"
        android:background="@drawable/corner6"
        android:gravity="center"
        android:orientation="horizontal" >

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="校园查询" />
    </LinearLayout>
        
    </LinearLayout>
</LinearLayout>
</LinearLayout>
```

### 参考链接

[android 怎么画一个圆角矩形_百度知道](http://zhidao.baidu.com/link?url=-hDkeReRRiNi22KN2Tql2ztPYOzX1J3aek83985R4mGLXoZpyi0Jw6fDvNT6M687z9AufAef7OvYNMSEool173Dscj6sUnvEPGJTEWVCSse)

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160511195417743)