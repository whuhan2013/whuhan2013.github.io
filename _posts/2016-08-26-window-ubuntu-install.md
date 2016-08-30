---
layout: post
title: 双系统安装详解
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

### 软件安装及其他       

**1、pycharm安装**               
要先安装JDK,速度挺慢的             
详细参见：[pycharm安装](http://blog.csdn.net/qq_33880788/article/details/51479564)           
  

**2、主文件夹改为中文**          

详细参考：[主文件夹改为中文](http://blog.csdn.net/l0605020112/article/details/20285239)

**3、Ubuntu使用Git**       

ubuntu中使用Git与显示隐藏文件，可参见：               
[Ubuntu中使用Git](http://www.cnblogs.com/fanyong/p/3424501.html)            
[显示隐藏文件](http://jingyan.baidu.com/album/6079ad0e84cd9728ff86dbc1.html)              
 

**4、sublime中无法输入中文**        

这个参照了很多方法始终无法解决，只能待以后了          

[解决Ubuntu下Sublime Text 3无法输入中文](http://www.jianshu.com/p/bf05fb3a4709)

[无法输入中文解决方案](http://blog.csdn.net/bleachswh/article/details/51674552)           


**5、Ubuntu系统下Chrome浏览器显示的汉字颜色很淡**      

解决方案：[chrome浏览器汉字颜色淡](https://www.zhihu.com/question/46303724)

**6、Ubuntu系统下修改hosts文件科学上网**                

- gedit /etc/hosts 打开hosts文件 然后下载好的hosts内容复制粘帖到/etc/hosts里 保存 
- 终端输入sudo systemctl restart NetworkManager 至此搞定

其中sudo systemctl restart NetworkManager还修复了我的网卡与声卡，真是神奇,也可以说是喜出望外，解决了长久的问题

参考：[ Ubuntu 16.04 经验积累和总结](http://blog.csdn.net/u013230444/article/details/51388211)


**7、Ubuntu系统下安装网易云音乐失败**         

Ubuntu下可以安装网易云音乐了，棒。不过安装时会缺少依赖，用下面这句命令即可。   

```
apt-get -f install
```

**8、Ubuntu 16.04中为Chromium、Chrome、Firefox安装Flash播放器插件**          

Ubuntu中除了B站外，其他网站不能看视频，是因为缺少Flash插件的原因，不过安装了之后，腾讯视频依旧不能看，其他的可以，不知道是为什么。       

具体步骤参见：[Ubuntu 16.04中为Chromium、Chrome、Firefox安装Flash播放器插件](http://www.linuxidc.com/Linux/2016-05/131098.htm)

**效果如下**        

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/android/p4.png)

