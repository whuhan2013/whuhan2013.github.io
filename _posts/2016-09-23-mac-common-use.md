---
layout: post
title: Mac使用入门
date: 2016-9-23
categories: blog
tags: [linux]
description: Mac使用入门
---


**mac常用快捷键**

```
全屏/退出全屏   ctr+command+F
切换到桌面      fn+f11
输入法切换      ctr+空格       
亮度           f1、f2
声音           f11、f12
复制、粘贴      command+c、command+v
搜索           command+f
翻页           上一行是Command+上箭头 上一页是Command+Fn+上箭头
sublime列操作   option+鼠标左键
pageDown、pageUp   fn+上下箭头
```

Mac不像Win那样，按住左上角关闭按钮就是退出程序，在Mac中‘关闭’的涵义只是关闭窗口，想要 退出程序 需要在菜单栏中或Dock右键“退出”程序

- 退出程序快捷键是⌘command+Q； 
- 关闭窗口快捷键是⌘command+W； 
- 最小化窗口快捷键是⌘command+M 


在Mac中自带了 最‘完美’的截图功能：

- 全屏截图：按下Command+Shift+3 ，截取全屏图像将以PNG透明格式保存到桌面上。      
- 区域截图：按下Command+Shift+4，鼠标变成十字准星，单击划出你想要的截图范围，自动保存到桌面上，想要取消截图按esc，想要移动选区按空格键即可   
- 窗口截图：按下Command+Shift+4后，再按下空格键，这时鼠标变成'小相机'，点击想要的部件，就可以以PNG透明格式存在桌面上      

不想要保存，按下ctrl就把截图保存在剪切板中

**mac修改终端背景颜色:可以在终端偏好设置中设置**


**mac中搭建android开发环境**

[Android Studio2.0 教程从入门到精通MAC版 - 安装篇](http://www.open-open.com/lib/view/open1466430392743.html)

[收集整理Android开发所需的Android SDK](https://github.com/inferjay/AndroidDevTools)


**mac高速下载百度云**     

在mac下的百度云客户端很烦，是同步盘，用浏览器下速度对太慢，最后采用aira2+浏览器插件实现百度云高速下载。

[Mac 百度云加速下载](http://xclient.info/a/6b6c46df-3e4f-1b17-ae30-0c8b49df92cc.html)


**mac效率提升**   

[提升你的 Mac 生产力](https://zhuanlan.zhihu.com/p/22673342)        
[Mac效率利器（一）系统管理篇](http://kaito-kidd.com/2016/09/13/Mac-edge-tools-system/)        
[Mac效率利器（二）开发篇](http://kaito-kidd.com/2016/09/26/Mac-edge-tools-dev/)


**mac下sublime代码格式化**       
安装HTML-CSS-JS Prettify: HTML-CSS-JavaScript 插件代码格式化      
快捷键 command+shift+H

sublime的zoomCoding           

可以快速生成大量重复代码，例如         
div.row>div.col-md-12*10  使用ctr+e快捷键之后，会生成十个div


**mac录屏工具**        
[Mac软件分享：上小巧实用的GIF格式录屏软件 LICEcap](http://www.cnblogs.com/emmet7life/p/4178599.html?utm_source=tuicool&utm_medium=referral)

**mac下安装tomcat**         

- 将下载的zip包解压到usr/local目录下        
- 将目录加到环境变量下，export PATH=${PATH}:/usr/local/apache-tomcat-8.5.6/bin，并用source .bash_profile激活     
- sudo chmod 755 xxx/bin/*.sh     (xxx表示你tomcat放至的路径) 回车
- startup.sh即可启动，shutdown.sh即可关闭      
- 设置用户名与密码：<user username="admin" password="1234" roles="manager-gui"/>     

参见：[mac下安装tomcat](http://blog.csdn.net/huyisu/article/details/38372663)


**mac下完全卸载postgresql**      

[mac下完全卸载postgresql的方法](http://blog.csdn.net/stk_tianwen/article/details/17757393)


**mac利用hosts文件科学上网**         

1. 从https://laod.cn/hosts/2016-google-hosts.html下载最新hosts文件    
2. 修改etc/hosts中的内容      
3. 终端中输入sudo killall -HUP mDNSResponder即可
