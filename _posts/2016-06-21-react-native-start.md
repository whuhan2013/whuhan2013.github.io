---
layout: post
title: ReactNative环境配置
date: 2016-6-21
categories: blog
tags: [React Native]
description: ReactNative环境配置
---

**参考链接**

[Windows系统安装React Native环境](http://www.lcode.org/%e5%8f%b2%e4%b8%8a%e6%9c%80%e8%af%a6%e7%bb%86windows%e7%89%88%e6%9c%ac%e6%90%ad%e5%bb%ba%e5%ae%89%e8%a3%85react-native%e7%8e%af%e5%a2%83%e9%85%8d%e7%bd%ae/)

[windows下React Native Android 环境搭建](http://blog.leanote.com/post/skuare520/841121f15ace)

[在Windows下搭建React Native Android开发环境](http://bbs.reactnative.cn/topic/10/%E5%9C%A8windows%E4%B8%8B%E6%90%AD%E5%BB%BAreact-native-android%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)


**碰到的问题** 

- react-native可能在cmd窗口提示不是内部或外部命令            
解决方法：在nodeJS command prompt下可以运行 
- 运行时卡在最后，程序是白屏              
解决方法：为应用程序添加悬浮窗权限  
- 下载时卡住了                 
解决方法: 更换rpm的数据源，用国内代理  


### IDE的使用 

**atom** 

atom安装插件会很慢，因为GFW的缘故，所以使用国内镜像 

apm config set strict-ssl false
修改registry到淘宝npm镜像:[为Atom配置前端开发环境](http://leftstick.github.io/tech/2015/07/01/setup-frontend-env-with-atom)

```
apm config set registry https://registry.npm.taobao.org
apm config set registry https://registry.npm.taobao.org
```

- 安装显示HTML控件Atom HTML Preview, apm install atom-html-preview
- 安装实现震撼编辑效果的插件: amp install activate-power-mode 



