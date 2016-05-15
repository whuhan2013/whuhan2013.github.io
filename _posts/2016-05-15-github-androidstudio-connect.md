---
layout: post
title: 如何用AndroidStudio导入github项目
date: 2016-5-15
categories: blog
tags: [github]
description: 如何用AndroidStudio导入github项目
---

### 转载  

最近一直在研究AndroidStudio，但是总会有这样那样的问题，特别是在github上看到一个很好地开源项目，想clone下来用用，就会出现很多蛋疼的问题，今天摸索着，结合一些大牛们的建议，轻轻松松的实现了，让那些蛋疼的问题交给AndroidStudio自己去解决吧。。。。 

第一步：              
你的电脑上首先要有git和AndroidStudio，没有的话赶紧下去吧，之前我的博客也有开发工具，这里我就当你有了，直接开始。

第二步：打开studio找到设置页面 

![](http://img.blog.csdn.net/20150515114521011)

将你安装的git路径放到第二步中，点击ok。 
第三步：你要有一个github 的账号，这里我就当你有了，接下来进行下面的配置 

![](http://img.blog.csdn.net/20150515115224930)

按照步骤一步步的来，点击Test，当出现这个界面的时候证明你的github和git已经配置成功 

![](http://img.blog.csdn.net/20150515114824654)

第四步：就要配置你要clone的项目地址了

![](http://img.blog.csdn.net/20150515120730901)

当你点击github，会这个样子 

![](http://img.blog.csdn.net/20150515115343086)

此处放你要clone的地址 
然后点击clone。 

![](http://img.blog.csdn.net/20150515115824584)

等一会会出现这个页面，然后点击yes 
会出现这个页面 

![](http://img.blog.csdn.net/20150515115931337)

有两个选项，第一个就是使用项目中默认的gradle版本，一个是使用你本地的gradle，我这里选择了第二个，这样的话就不用再去下载了，下载的话，你懂得。然后点击ok，等待就好。 


![](http://img.blog.csdn.net/20150515120134437)


这个样子证明你已经成功了 
根据个人喜好，选择一下就好，一个在当前窗口代开此项目，一个是在一个新窗口打开此项目 
当你成功之后，在你的工具栏VCS当中 

![](http://img.blog.csdn.net/20150515120420191)

就会出现对git的一系列操作，pull，push，add都可以在这里面进行了。 
这两个一个是pull，一个是push，也可以 

![](http://img.blog.csdn.net/20150515120518032)

### 参考链接

[如何用AndroidStudio导入github项目-布布扣-bubuko.com](http://www.bubuko.com/infodetail-807716.html)





