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

