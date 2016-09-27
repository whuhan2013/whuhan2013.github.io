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


**淘宝npm镜像**

来源：https://cnodejs.org/topic/4f9904f9407edba21468f31e
 
镜像使用方法（三种办法任意一种都能解决问题，建议使用第三种，将配置写死，下次用的时候配置还在）:
 
1.通过config命令           
 npm config set registry https://registry.npm.taobao.org             
npm info underscore （如果上面配置正确这个命令会有字符串response）      

2.命令行指定            
 npm --registry https://registry.npm.taobao.org info underscore            
3.编辑 ~/.npmrc 加入下面内容              
 registry = https://registry.npm.taobao.org                 


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


**错误&问题** 

- reload JS变成reload，并且不能动态更新   
解决方法：升级npm即可，这个卡了好久啊，竟然是这里出了问题。 
- npm install出错  
解决方法：同上，升级npm解决


**init时指定特定版本** 

```
mkdir  [Project Name] && cd $_
npm i react-native@version --save
react-native upgrade
```

**References**

[创建 React-Native 工程时，如何指定特定的 React-Native 版本 - 简书](http://www.jianshu.com/p/646c5fbd9659)


**Mac下搭建react native环境**     

mac下搭建react native环境比在windows下快多了，少了很多坑，十分钟就解决了，而且ios模拟器也不卡，真的是很棒，具体过程可以参见：

[mac下搭建reactnative环境](http://www.lcode.org/%E3%80%90react-native%E5%BC%80%E5%8F%91%E3%80%91react-native-for-android%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE%E4%BB%A5%E5%8F%8A%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%AE%9E%E4%BE%8B/)

**问题**    

**运行android项目时要配置ANDROID_HOME**   

vim .bash_profile编辑

```
export ANDROID_HOME=~/Library/Android/sdk
export PATH=${PATH}:~/Library/Android/sdk/platform-tools/
export PATH=${PATH}:~/Library/Android/sdk/tools/
```

**使用sublime3开发react native**     

- ReactJS : 支持React开发，代码提示，高亮显示 ，介绍地址：点我
- Emmet ：前端开发必备。
- Terminal : 在sublime中打开终端并定位到当前目录，神器，mac下的快捷键为：command+shift+T
- react-native-snippets：react native 的代码片段

参见：[Sublime开发react native](http://www.jianshu.com/p/2ddfff095e90]

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/new/p1.png)




