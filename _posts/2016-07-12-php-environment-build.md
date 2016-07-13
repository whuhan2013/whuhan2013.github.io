---
layout: post
title: PHP环境搭建
date: 2016-7-12
categories: blog
tags: [PHP]
description: PHP
---


在搭建PHP环境的过程中遇到不少坑，在这里记录下来，方便自己复习以及后来者。   

### Apache安装过程

**下载安装过程参考**         
[如何从Apache官网下载windows版apache服务器_百度经验](http://jingyan.baidu.com/article/29697b912f6539ab20de3cf8.html)            
[Apache服务器最新版下载、安装及配置（win版）_百度经验](http://jingyan.baidu.com/article/d8072ac47baf0eec95cefdca.html)

- httpd -k install安装服务    
在安装服务的过程中可能会报ServerRoot must be valid directory的错误，就将ServerRoot修改为我们的安装地址，注意斜杠的
方向，形如ServerRoot "D:\amp\apache"  ,在注册成功后，在Apache Monitor中打开service. 参考：[Apache报ServerRoot must be a valid directory_百度经验](http://jingyan.baidu.com/article/915fc41491c68751384b2040.html)     

- 80端口被pid为4的NT Kernel占用         
解决方法：[80端口的烦恼：[3]清除NT Kernel占用80端口_百度经验](http://jingyan.baidu.com/article/f96699bbca15a1894e3c1bc4.html)

- apache服务的卸载     
sc delete apache     其中apache是Apache服务器的服务名


**安装成功效果如下**       
![](http://g.hiphotos.baidu.com/exp/w=480/sign=2c544a5beb50352ab16124006342fb1a/4ec2d5628535e5ddf1b6e03374c6a7efcf1b6247.jpg)


### php安装   

详细过程请参考：[最新php环境搭建_百度经验](http://jingyan.baidu.com/article/154b46315242b328ca8f4101.html)

**安装完成后利用phpinfo查看安装情况**  

```
<?php
phpinfo();
?>
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p1.png)

**当在phpinfo中可以看到mysql则说明mysql配置成功**    
当看不到时，或者访问mysql时发生500错误，解决方法为将 php_mysql.dll    ,libmysql.dll 拷贝到system32下,将php,php ext加入到环境变量中，然后再重启试试。        

**mysql测试代码**       

```
<?php 
$link=mysql_connect('localhost','root','root'); 
if(!$link) echo "FAILED!"; 
else echo "SUCCESS!"; 
mysql_close(); 
?>
```


### 虚拟主机配置          
通常情况下，一个web服务代理一个网站，但是有时候我们需要用一台服务器代理多个网站。这个就是基于域名的虚拟主机技术。

我们可以直接在httpd.conf当中进行配置，也可以使用extra中的httpd-vhosts.conf配置，建议使用第二种方式。

首先要开启vhost.conf配置
![](http://img.blog.csdn.net/20140623142317546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGtncmF5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中*表示所有的ip地址，如果是一个具体的ip，你可以写上这个ip，但建议使用*，80指的是端口。

接下来，要单独的配置具体的域名，通过 VirtualHost 指令段，其参数必须和NameVirtualHost

而且在指令段中必须包含ServerName 和 DocumentRoot
![](http://img.blog.csdn.net/20140623142352125?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcGtncmF5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 访问权限配置 

Apache通过配置项 <Directory 所需配置的目录>来实现的

**参考：**[Apache配置某个目录的权限_百度经验](http://jingyan.baidu.com/article/219f4bf7ff4fe6de442d3880.html)

[php虚拟主机配置、访问权限配置、分布式文件配置 - pkgray的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/pkgray/article/details/33730995)



