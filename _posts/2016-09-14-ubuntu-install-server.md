---
layout: post
title: Ubuntu安装服务器 
date: 2016-9-14
categories: blog
tags: [linux]
description: Ubuntu安装服务器 
---


### tomcat的安装

安装Tomcat需要首先安装配置JDK

#### Java的安装  

在Ubuntu和Linux Mint上安装Oracle JDK

使用下面的命令安装，只需一些时间，它就会下载许多的文件，所及你要确保你的网络环境良好：

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
sudo apt-get install oracle-java8-set-default
```

**详细可参见：**[怎样在Ubuntu 14.04中安装Java](http://www.linuxidc.com/Linux/2014-09/106445.htm)

**注意：**安装路径在/usr/lib/jvm中

#### 安装tomcat

从官网下载tar.gz版的包解压，再移动到opt目录下

打开编辑启动的脚本文件
sudo vi ./bin/startup.sh
打开这个文件，要往里面写入jdk路径。

```
JAVA_HOME=/usr/lib/jvm/jdk1.8.0_31
JRE_HOME=${JAVA_HOME}/jre
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
TOMCAT_HOME=/opt/apache-tomcat-8.0.17
```

**注意：**不要直接拷贝，要按自己之前的jdk路径来。注意文件夹的名字。按自己的来。

**注意：**不是直接放在放在整个文件的最后。我的startup.sh文件的最后一行是：
exec "$PRGDIR"/"$EXECUTABLE" start "$@"

![](http://img.blog.csdn.net/20150124182627078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FybG9zMTk5Mg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

启动tomcat：

```
 sudo ./bin/startup.sh
```

这样，就可以启动tomcat 了,关闭这个文件，和startup.sh文件一样，即启动tomcat时运行的文件一样，需要添加jdk路径，方法是一样的


**详情可参见：**[Ubuntu 14.04 安装jdk，tomcat](http://blog.csdn.net/carlos1992/article/details/43085897)


### apache安装 

apache的安装比较简单，可以直接几句命令搞定。

**详情参见：**[手把手教你在ubuntu上安装apache和mysql和php](http://blog.csdn.net/guaikai/article/details/6905781)


**Ubuntu下启动/重启/停止apache服务器**

```
Task: Start Apache 2 Server /启动apache服务
# /etc/init.d/apache2 start
or
$ sudo /etc/init.d/apache2 start
Task: Restart Apache 2 Server /重启apache服务
# /etc/init.d/apache2 restart
or
$ sudo /etc/init.d/apache2 restart
Task: Stop Apache 2 Server /停止apache服务
# /etc/init.d/apache2 stop
or
$ sudo /etc/init.d/apache2 stop
```

**ubuntu下安装wordpress**

[Ubuntu下搭建wordpress环境](http://www.jianshu.com/p/26d9e752994e)


**Ubuntu 14.04下 Apache修改网站根目录及默认网页**

```
修改根目录： 
在 /etc/apache2/sites-available 中修改 000-default.conf 
中的DocumentRoot /var/www/ 修改为想要的目录 
比如：DocumentRoot /var/www/html/mainpage


接下来重启apache,sudo apache2ctl -k restart 即可

修改默认网页： 
修改/etc/apache2/mods-available/dir.conf中的内容 
原来是：

<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm 
</IfModule>

添加上想要的/wordpress就行啦~

<IfModule mod_dir.c>
    DirectoryIndex index.html index.cgi index.pl index.php index.xhtml index.htm /wordpress
</IfModule>
```

**参考：**[Ubuntu 14.04下 Apache修改网站根目录及默认网页](http://www.linuxidc.com/Linux/2015-07/120724.htm)