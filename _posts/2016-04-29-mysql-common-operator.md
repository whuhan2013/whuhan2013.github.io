---
layout: post
title: mysql常用操作
date: 2016-4-29
categories: blog
tags: [数据库]
description: mysql常用操作
---

### 本文主要包括数据库常用操作  

### 连接Mysql 

- 连接到本机上的MYSQL。      
首先打开DOS窗口，然后进入目录mysql\bin，再键入命令mysql -u root -p，回车后提示你输密码.注意用户名前可以有空格也可以没有空格，但是密码前必须没有空格，否则让你重新输入密码。

如果刚安装好MYSQL，超级用户root是没有密码的，故直接回车即可进入到MYSQL中了，MYSQL的提示符是： mysql>

- 连接到远程主机上的MYSQL           
假设远程主机的IP为：110.110.110.110，用户名为root,密码为abcd123。则键入以下命令：  

```
    mysql -h110.110.110.110 -u root -p 123;（注:u与root之间可以不用加空格，其它也一样）
```

- 退出MYSQL命令： exit （回车）  

### 修改密码 

格式：mysqladmin -u用户名 -p旧密码 password 新密码

- 给root加个密码ab12。      
首先在DOS下进入目录mysql\bin，然后键入以下命令   

```
    mysqladmin -u root -password ab12
```         

注：因为开始时root没有密码，所以-p旧密码一项就可以省略了。

- 再将root的密码改为djg345     

```
    mysqladmin -u root -p ab12 password djg345
```  




### 创建一个数据库与表

```
create database struts;
use struts;
create table user(
	id varchar(100) primary key,
	username varchar(100) not null unique,
	nick varchar(100),
	password varchar(100),
	sex varchar(10),#0 女 1男
	birthday date,
	education varchar(100),#研究生 本科 专科
	telephone varchar(100),
	hobby varchar(100),#体育,旅游,吃饭
	path varchar(100),#上传文件保存的路径
	filename varchar(100),#uuid_oldfilename.jpeg
	remark varchar(255)
);
```  

### 删除数据库与表  

- 删除数据库  

```
drop database struts;
```

- 删除表  

```
 mysql> drop table MyClass;
```

### 重启mysql  

开始->运行->cmd
停止：net stop mysql
启动：net start mysql
前提MYSQL已经安装为windows服务



### 参考链接  

[Mysql命令大全 - 宁静.致远 - 博客园](http://www.cnblogs.com/zhangzhu/archive/2013/07/04/3172486.html)



