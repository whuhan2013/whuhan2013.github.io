---
layout: post
title: IOS环境搭建
date: 2016-9-27
categories: blog
tags: [IOS]
description: IOS
---


ios的开发环境比android要简单不少，只需要下载xcode即可，只不过需要mac电脑，iphone手机，开发者账号而已，算下来应该要小两万吧。

> Xcode 7 包含您为 iPhone、iPad、Mac 和 Apple Watch 创建酷炫 app 所需的全部资源。Swift 编程语言已完成相应更新，现在速度比以往任何时候更快，并提供诸多精彩功能，让代码读取和写入变得更加容易。此外，借助新增的 Playground，您可以尝试新的 API，也可以使用嵌入资源、其他源代码和富文本注释编写极佳的互动文档。Xcode 的用户界面测试功能甚至可以记录您 app 的运作方式，并为您生成测试。

xcode非常方便，在xcode下创建的模拟器也一点不卡，比android流畅多了。


**IOS真机调试**     

以前ios的真机调试需要开发者账号，所幸从xcode7之后已经不需要了，只需要普通apple id即可。参见：[Xcode 7：无需99刀也能在真机上测试App](http://www.cocoachina.com/ios/20150611/12123.html)


**问题**

1.  Error: An App ID with identifier "*" is not avaliable. Please enter a different string.

错误原因是这个bundle ID已经被别人提前占用了，bundle ID必须是唯一的。
解决办法当然是修改你的bundle ID 了。

2. don't load library,cannot find suitable image     

ios项目在模拟器上测试时均无此错误，在真机上测试时有此现象，测试发现react native项目与oc项目均无此问题，swift项目有这个问题，此问题尚未解决。


