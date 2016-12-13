---
layout: post
title: Appium入门
date: 2016-11-9
categories: blog
tags: [IOS]
description: Appium入门
---


Appium作为一个开源的、跨平台的自动化测试工具，适用于测试原生或混合型移动App。  Appium的核心是一个web服务器，他使用WebDriver json wire协议，来驱动系统的UIAutomation库。WebDriver Json wire协议的Server端采用node.js封装了iOS UI Automation的接口，提供提供出一套RESTFul web service的接口，这样Client端以HTTP请求获得操纵UI的能力。

说到底，真正执行测试的还是 UIAutomation，Appium只是封装或解释了UIAutomation的执行脚本，作为UIAutomation和被测试APP的中间层传递消息。

appium的优缺点

优点：    
（1） 跨平台 - appium可以很好的融合在addroid和iOS系统之间     
（2） 支持多种语言 - 支持各种语言对appium的脚本编写，但是好像oc的支持不太好     
（3） 不依赖源代码 - 不用依赖于源码的支持，这是一个很突出的亮点                   
（4） 开源 - 这个说主要也不算主要，因为appium是给予UIAutomaiton之上的，而UIAutomation不是开源的    

缺点：         
（1） 环境配置较繁琐 - 配置及其繁琐，而且问题较多，需要你耐心的就一点点解决，iOS版本更为严重      
（2） 不支持自定义控件         
（3） UIWebView的状态不可访问                             
（4） 无法脱机跑，需要连着Mac机器 - 这是iOS自动化框架共有的硬伤          
（5） 支持系统效率慢 - 这是我认为这个框架比较严重的伤，由于不是苹果公司自有的框架，在支持上总慢一两个月，所以很多人在适配新系统的时候比较头疼
Appium是由client和server组成，client提供多种语言的API，这些API是对WebDriver的扩展和封装，利用这些API就可以快速编写测试用例；client和server间通过符合Mobile JSON Wire Protocol的http请求进行交互。  

Appuim还提供一个第三文的工具Appium Inspector

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
npm install -g ios-deploy
进入nodumodule目录下，把webdriveragent与webdriveragentrunner都签名
```

**参考链接**        
[https://github.com/appium/appium/issues/6867](https://github.com/appium/appium/issues/6867)       
[appium-xcuitest-driver](https://github.com/appium/appium-xcuitest-driver/#external-dependencies)       
[Ios 10 with xcode 8 issue](https://discuss.appium.io/t/ios-10-with-xcode-8-issue/12267)       

**现存的问题**       
[XCUITest - XCode 8 - iOS 10 Tests Are Running Very Slow](https://github.com/appium/appium/issues/7284)     


