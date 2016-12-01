---
layout: post
title: Appium入门
date: 2016-11-9
categories: blog
tags: [IOS]
description: Appium入门
---

Appium的使用，主要有四个方面的因素：

**一，Appium Server**

1. Appium Server的安装

前提：已经安装node.js&npm        

可以能过命令行安装 

```
#sudo npm install －g appium　　//加上sudo以防Permission的问题

#npm install wd　　//这个还不清楚有什么影响？？
```

也可以在官网上下载下来安装，下载下来要快很多          

**二，Selenium WebDriver**

因为是Python版，可以在github上下载

https://github.com/appium/python-client

进入目录下安装

#sudo python setup.py install　　//sudo依旧是解决Permission的问题

这样，WebDriver就安装成功了。

**三，要测试的app**

测试的是appium提供的TestApp

首先，我们需要用xcode编译这个app

```
#cd appium

#cd sample-code/apps/TestApp

#xcodebuild -sdk iphonesimulator　　//为了防止iphonesimulator和设置的冲突，没有注明iphonesimulator的版本
```

如果看到** BUILD SUCCEEDED **，这个TestApp就build成功了。   
sample-code下载地址：[https://github.com/appium/sample-code](https://github.com/appium/sample-code)

**四，Automation Scripts**

自动化脚本，也是用appium提供的，在appium目录下可以找到

```
#cd appium

#cd sample-code

#cd examples/python

#python ios_simple.py　　　　　　//执行测试脚本

```

此时，iOS的模拟器就会打开，开始执行simple.py的测试脚本了！          

**参考链接**       
[Appium探索——Mac OS Python版](http://www.cnblogs.com/enjoytesting/p/3513637.html)      
[手把手教你appium_ios第一个例子](http://blog.csdn.net/testingba/article/details/23829425?utm_source=tuicool&utm_medium=referral)     



**ios真机上运行**       

可以通过appium inspector来探测元素，但首先要连上真机，要注意以下几点      

- 将ip修改为127.0.0.1    
- 直接通过ideviceinstaller -u [Your device's UID] -i [Path to your debug build]安装可能会导致权限问题，所以通过xcode安装    
- 通过udid与包名找到相应app   
- 通过sudo chmod -R 777 /var/db/lockdown/修改ideviceinstaller权限

**参考:**   
[WebDriverException: An unknown server-side error occurred while processing the command. Original error: Removing {appId} failed](http://stackoverflow.com/questions/39522679/webdriverexception-an-unknown-server-side-error-occurred-while-processing-the-c)   
[appium+Python真机运行测试demo的方法](http://www.cnblogs.com/Nefeltari/p/5603163.html)


**升级到appium1.6**      

```
npm install -g appium    
brew install carthage
```


