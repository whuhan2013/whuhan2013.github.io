---
layout: post
title: 编码问题总结
date: 2016-6-6
categories: blog
tags: [错误&问题]
description: 编码问题总结
---


### 本文主要包括以下内容  

1. xutils请求乱码 


### xutils请求乱码  

**解决方案**  

- 第一：查看自己的mysql的编码
- 第二：更改Latin1为utf-8
- 第三：如果你使用的tomcat进行通信的话，由于tomcat的默认编码是iso8859-1。
- 第四：在传输中文时用URLEncoder
- 第五：经过上面的步骤，存储到mysql的中文数据还是乱码，这是你的数据库建立有问题了。


**参考链接**

[在使用xutils时post请求传递中文到服务端Mysql数据库出现中文乱码](http://blog.csdn.net/u014131921/article/details/51346017)



