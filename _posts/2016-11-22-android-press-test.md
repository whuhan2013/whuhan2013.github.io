---
layout: post
title: Android压力测试
date: 2016-11-22
categories: blog
tags: [android]
description: Android压力测试
---

本文主要包括以下内容    
1、Monkey工具     
2、adb命令实现手机上的monkey工具的控制与使用，同是可以使用adb在手机上完成安装与卸载。     
3、monkey Script可以帮助完成重复的安装过程，例如搜索100次，monkey完成的是随即操作时无法完成重复性操作。    
4、monkeyRunner提供了三大aba。      

![](http://img.mukewang.com/5801e7b90001b4de12800720.jpg)  

**压力测试步骤**     

1.在手机开发者选项中，将USB调试连上    
2.确认连接 adb devices                   
3.安装测试APP  adb  install  package.apk               
4.发送压力测试指令   adb  shell  monkey  1000（具体事件数）       
5.获取APP包名  adb  logcat｜grep  START                
6.adb shell monkey-p package（包名）  1000（具体事件数）      

**monkey高级参数** 

```
adb shell monkey --throttle <milliseconds> 延时    
seed 参数：指定随机生成数的seed的值
adb shell monkey -s＜seed> 1000＜event-count>
设定触摸事件百分比    
--pct-touch 100  触摸事件100%
--pct-motion 100%  动作事件100%
系统导航事件（设定系统导航事件百分比，HOME、BACK拨号及音量键）
adb shell monkey --pct-syskeys<percent>
```

![](http://img.mukewang.com/580da923000118a712800720.jpg)    

![](http://img.mukewang.com/58107200000140ae12800720.jpg)

![](http://img.mukewang.com/5816e9b60001d0c412800720.jpg)

Monkey Runner 与monkeyscript差别是 runner 在测试过程中可以实现截屏操作

