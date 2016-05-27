---
layout: post
title: Android主题换肤实现
date: 2016-5-27
categories: blog
tags: [Material Design]
description: Android主题换肤实现
---   

**本系列文章主要是对一个Material Design的APP的深度解析，主要包括以下内容**


- 基于Material Design Support Library作为项目整体框架。对应博文：Android Material Design 兼容库的使用详解

- RecyclerView的万能适配器。对应博文:打造一个RecyclerView的万能适配器-减少你的代码冗余

- 高仿QQ的自定义View。对应博文：Android自定义View之高仿QQ健康

- 主题换肤功能。对应博文：Android主题换肤 无缝切换


### 效果图

![](https://camo.githubusercontent.com/e987bca637da4a687bd29989eb3ef40ff2a896af/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f3632333530342d306338613063373264336131373365642e6769663f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970)



### Android主题换肤


### 源码解析参见 


[Android主题换肤 无缝切换 - 简书](http://www.jianshu.com/p/af7c0585dd5b)


### 使用 

在使用的时候一定要记得要Activity要去继承于SkinBaseActivity，Fragment要继承于SkinBaseFragment，Application要继承于SkinBaseApplication。当然把这个框架做为你的项目依赖项肯定是必不可少的。为了Demo的简单，这里我只使用了下面三个颜色作为可以换肤的资源，当然如果你想要使用drawable文件也是可以办到的，前提是你一定要把这个Demo看懂。

![](http://upload-images.jianshu.io/upload_images/623504-08dec49645f73d4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


来看一个布局文件

![](http://upload-images.jianshu.io/upload_images/623504-24783a75958a40d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其中
xmlns:skin="http://schemas.android.com/android/skin"
是我们自定义的，在SkinConfig有。
我们只需在有皮肤更改需求的View中加入skin:enable="true" 就OK了。

再来看看MainActicvity的部分代码

![](http://upload-images.jianshu.io/upload_images/623504-da99c73eca73106a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这里就是动态的添加有皮肤更改需求的View。

上面就介绍完了在布局文件中使用方法和在代码中使用方法。

我们应该怎么去换肤呢？很简单，只需调用SkinManager的load方法就可以了，把皮肤路径传进去就可以了，我的这个Demo为了简单起见，没有做在线换肤的功能，只是在本地提供了可以更换的皮肤，看到这里我相信你对怎样在线换肤已经有想法了

![](http://upload-images.jianshu.io/upload_images/623504-8818a549a834dc96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


最最后我们来看看怎么去开发皮肤包。其实这个是最简单的，皮肤包实际上就是一个基本的Android项目，里面不包含类文件，只有资源文件。这里只需注意 这里的资源文件名字一定要和原项目中的相同，并且只用包含那些在皮肤更改时需要改变的那些就行了！例如我的这个Demo就只是简单对上面的三种颜色做了简单的切换。开发了棕色和黑色两款皮肤，所以资源文件中只有三个color的值，开发完成之后我们需要将其打包成apk文件，为防止用户点击安装，我们将其后缀改成了skin，这样做也具有标识性。如果还是不太清楚可以直接去源码中查看。

### 源代码

[代码下载](https://github.com/burgessjp/MaterialDesignDemo)


### 参考链接

[Android主题换肤 无缝切换 - 简书](http://www.jianshu.com/p/af7c0585dd5b)
