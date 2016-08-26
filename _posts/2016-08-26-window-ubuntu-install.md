---
layout: post
title: 双系统安装过程总结
date: 2016-8-26
categories: blog
tags: [linux]
description: 
---

### 工具/原料   

- UltraISO         
- Ubuntu 16.04 LTS安装镜像         
- 2G以上U盘 1个       
- easyBCD           

### 准备工作

**使用UltraISO制作U盘启动盘**        
[Ubuntu 14.04 LTS U盘启动安装教程_百度经验](http://jingyan.baidu.com/article/eb9f7b6d8536a8869364e813.html)

**系统空间准备**    

右键点击我的电脑》管理》磁盘管理,选择一个空间较大的盘,右键选择压缩卷,大小自己分配。         
详见：[Win7下U盘安装Ubuntu14.04双系统步骤详解_百度经验](http://jingyan.baidu.com/article/76a7e409bea83efc3b6e1507.html)

**安装过程**

详见：[Win10和Ubuntu16.04双系统安装详解 - 简书](http://www.jianshu.com/p/16b36b912b02)

### 碰到的问题   

**1、使用easyBCD修改系统启动项更改**    
安装完Ubuntu之后,从Win7启动来做引导可以让我们更自由的选择是否需要Ubuntu系统,以后不想继续使用Ubuntu系统可以直接在Win7里面将Ubuntu的分区格式化,而不会影响Win7操作系统,这也是与从Ubuntu启动最大的好处(若选择从Ubuntu里引导启动Win7将来容易出问题,尤其是Ubuntu出问题的时候)。

但这里可能会产生很多问题，所以如果不是特别熟悉的话就不用更改了。

**2、开机直接就进到Ubuntu 了，怎么能进入Windows?**            

装好后没有，在终端执行sudo update-grub后重启就有啦

**3、重装系统后，原系统时间混乱**                          
修改Windows注册表，让其将硬件时间识别为UTC时间，点击开始菜单–运行–CMD（Win7用户需使用管理员身份运行）–输入或粘贴以下命令：
Reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1  之后重启即可解决    

参考:[PC装mac双系统后，win7的系统不对，相差8小时](http://zhidao.baidu.com/link?url=L0Uak8A372nwwA0tpkToEMvRABtSNzY80lhmtpbzV4KB8A3Ptd18MXKW2Jmi8PZcVSwUPm4L8P7BGV0BMKo_zE16WqwN3j15T_tldLnCtIe)

**4、此windows副本不是正版的问题**        

在CMD中输入“SLMGR -REARM”（注意中间有空格）按回车键，当出现Windows script host窗口后点击“OK”，然后重启电脑即可。

参考：[小技巧教你解决此windows副本不是正版的问题_百度经验](http://jingyan.baidu.com/article/7c6fb42869452380642c9027.html)

**效果如下**        

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/android/p4.png)

