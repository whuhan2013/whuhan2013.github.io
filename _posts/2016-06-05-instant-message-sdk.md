---
layout: post
title: 环信SDK集成
date: 2016-6-5
categories: blog
tags: [android]
description: 环信SDK集成
---

### 利用环信SDK可以实现即时通讯，但在集成的过程中碰到了不少的坑。

**注意**

选择项目路径，这里以最新版环信demo为例
注意：环信的ChatDemoUI这个demo里边因为研发的同事为了照顾老版本的AndroidStudio使用者，已经用eclipse生成了build.gradle文件，所以如果要导入新版AndroidStudio请把build.gradle删除


![](http://lzan13.qiniudn.com/blog/uploads/images/2015/11/tupian002.png)




**参考链接**  

[关于新版AndroidStudio导入环信Demo的一些注意事项](http://melove.net/Develop/Android/android-easemob-demo-import/2015-11-10013.html)



### 导入环信新版EaseUI库问题

![](http://lzan13.qiniudn.com/blog/uploads/images/2015/11/tupian012.png)


出现问题的原因的大致因为EaseUI默认引入的v4包的版本20.0.0，但是大家的开发环境不同，SDK版本以及编译器和support库版本不同，会出现错误；



**解决办法：**         
这个时候就去点击项目设置，选中EaseUI把sdk版本设置成和build.gradle里一样的版本就行了，如果过低，建议更新Android SDK Tool，还不行，就把自己的项目的SDK版本和EaseUI都设置成一样，v4库也设置成一样



### 参考链接

[AndroidStudio导入环信新版EaseUI库问题](http://melove.net/Develop/Android/android-androidstudio-import-easeui/2015-11-10006.html)



### 这样之后还是会报错  

```
Error:(12, 0) Error: NDK integration is deprecated in the current plugin.  
Consider trying the new experimental plugin.  
For details, see http://tools.android.com/tech-docs/new-build-system/gradle-experimental.  
Set "android.useDeprecatedNdk=true" in gradle.properties to continue using the current NDK integration.
```


**解决方法**

[android.useDeprecatedNdk=true 添入工程根目录下的新建 gradle.properties 文件-andersonyan-ChinaUnix博客](http://blog.chinaunix.net/uid-26000296-id-5521954.html)

### 最后可能还要添加配置NDK的路径


**Demo效果如下**

![这里写图片描述](http://img.blog.csdn.net/20160605162231055)



**参考链接**

[使用插件化方式快速集成环信即时通讯_百度经验](http://jingyan.baidu.com/article/9113f81b0cbce22b3214c72b.html)