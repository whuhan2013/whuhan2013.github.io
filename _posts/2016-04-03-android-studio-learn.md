---
layout: post
title: Android Studio安装与配置
date: 2016-4-3
categories: blog
tags: [android]
description: Android Studio安装与配置
---

  谷歌已经停止支持eclipse开发android了，转向android studio是大势所趋，笔者由于电脑配置的原因，
  以前迟迟不愿意向android studio，现如今因为开始学习material design,不得不转向android studio了，
  费了一番功夫，终于把环境配置好了。

### 下载与安装android studio
  参考链接：
  [android studio安装使用教程（详细图文教程）_百度经验](http://jingyan.baidu.com/article/ad310e80a9328a1849f49e30.html)

### 注意，因为笔者先前已经安装了SDK,所以不用再下载安装SDK了，省却了不少时间，不然的话，可以在[inferjay/AndroidDevTools: 收集整理Android开发所需的Android SDK、开发中用到的工具、Android开发教程、Android设计规范，免费的设计素材等。](https://github.com/inferjay/AndroidDevTools),下载相关资源，也可以使用代理，使用sdk manager下载


### 笔者在导入已经存在的项目的时候，总是会卡在gradle这里，这是因为在build.grandle里面指定了gradle的版本，如果
与我们的版本不一致的话，则会去下载，但国内下载gradle巨慢，所以我们可以\gradle\wrapper文件夹下的gradle-wrapper.properties文件中的指定的gradle指定为我们自己的。

### 由于真机的SDK版本有限，我们也可以使用号称最快模拟器的Genymotion来开发

### 参考链接:  
[安卓模拟器Genymotion安装使用教程详解_百度经验](http://jingyan.baidu.com/article/3ea51489e7d8bd52e61bba36.html)  
[使用Android Studio搭建Android集成开发环境（图文教程） - 生命壹号 - 博客园](http://www.cnblogs.com/smyhvae/p/4022844.html)

### 安装好了Genymotion后需要下载模拟器，但常常会出现Failed to deploy virtual device，原因是因为服务器在国外的原因。。。所以我们可以通过下载离线ova文件来解决。   

### 参考链接：  
[Failed to deploy virtual device_genymotion吧_百度贴吧](http://tieba.baidu.com/p/4297513918)

### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160403153655621)












