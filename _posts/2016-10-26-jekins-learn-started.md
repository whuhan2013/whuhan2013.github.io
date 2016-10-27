---
layout: post
title: Jenkins学习入门
date: 2016-10-26
categories: blog
tags: [java]
description: Jenkins学习入门
---


**安装**       
略

**Mac环境中Jenkins的停止和启动命令**           

```
启动

sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist

停止

sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
```


**在mac中找到jdk地址**            
[MAC中jdk的目录](http://www.cnblogs.com/JinUzuki/articles/2130321.html)     


**遇到的问题**        

**1.adb command not found**          
由于jenkins会创建一个新用户，导致原有的环境变量不能使用，所以要在jenkins用户下也安装adb       

**参考：**[jenkins+github+gradle 实现android自动化打包全攻略](http://www.jianshu.com/p/9caab25d2cf1/comments/2874371)

