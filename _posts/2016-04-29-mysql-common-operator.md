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


### 实例  

Estore电子商城 --- 知识的整合,加入一些真实开发中的思想 面向切面编程 利用注解加本地线程的方式实现事务管理  
    用户注册(发送激活邮件/JS前台实现数据校验/验证码)        
    用户激活             
    用户登录(记住用户名/30天内自动登陆)             
    用户注销                    
    添加商品(文件上传)                  
    查看商品列表                   
    查看商品详情                       
    加入购物车(Cookie *session 数据库)                      
    增删改查购物车              
    生成订单(多表设计)                    
    订单查询(多表查询)                                                      
    订单删除(事务管理/注解+本地线程+动态代理实现事务管理,AOP--面向切面编程)               
    在线支付(体验调用第三方接口实现特定功能)             
    销售榜单下载(利用程序生成excel数据/文件下载)                      
    
    权限过滤器                 
    全站乱码过滤器                                    
    
    用户:id 用户名 密码 昵称 邮箱 激活状态 激活码 角色 注册时间        
    商品:id 商品名称 商品种类 商品库存数量  商品单价 图片url 描述信息           
    订单:id 下单时间 收货地址 支付状态 订单金额 用户编号(外键)            
    订单项: 订单id  商品id  购买数量              
     
    用户 1 -- * 订单            
    商品 * -- *订单             
    
    create database estore;                
    
    用户：
			create user estore identified by 'estore';            
     
    授权：              
			grant all on estore.* to estore;                  
    
    use estore;     


```        
    create table users ( 
			id int primary key auto_increment,
			username varchar(40),
			password varchar(100),
			nickname varchar(40),
			email varchar(100),
			role varchar(100) ,
			state int ,
			activecode varchar(100),
			updatetime timestamp
	        );



		 create table products(
			id varchar(100) primary key ,
			name varchar(40),
			price double,
			category varchar(40),
			pnum int ,
			imgurl varchar(100),
			description varchar(255)
			);



		create table orders(
			id varchar(100) primary key,
			money double,
			receiverinfo varchar(255),
			paystate int,
			ordertime timestamp,
			user_id int ,
			foreign key(user_id) references users(id)
		);

		create table orderitem(
			order_id varchar(100),
			product_id varchar(100),
			buynum int ,
			primary key(order_id,product_id), #联合主键,两列的值加在一起作为这张表的主键使用
			foreign key(order_id) references orders(id),
			foreign key(product_id) references products(id)
		);
```
        
      



### mysql数据库要按当天、昨天、前七日、近三十天、季度、年查询

天笔者做一个销售系统查询，要按当天、昨天、前七日、近三十天、季度、年查询，我用的是php+mysql制作，常用的时间查询条件mysql语句，笔者在本地测试并且使用了，没有错！分享给大家了 查询今天 sql语句 

```
select * from 表名 where to_days(时间字段名) = to_days(now());  
```
   
查询昨天 sql语句 

```
SELECT * FROM 表名 WHERE TO_DAYS(NOW())-TO_DAYS(`时间字段名`) = 1   
```

查询7天 sql语句 

```
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(时间字段名)    
```

查询近30天 sql语句 

```
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)   
```

查询本月 sql语句 

```
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, ‘%Y%m’ ) = DATE_FORMAT( CURDATE( ) , ‘%Y%m’ )    
```

查询上一月 sql语句 

```
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , ‘%Y%m’ ) , date_format( 时间字段名, ‘%Y%m’ ) ) =1  
```

同时，再附上 一个 mysql官方的相关document    

```
#查询本季度数据 
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(now());

 #查询上季度数据 
select * from `ht_invoice_information` where 
QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER)); 

#查询本年数据 
select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW()); 

#查询上年数据 
select * from `ht_invoice_information` where 
year(create_date)=year(date_sub(now(),interval 1 year));   

查询当前这周的数据  
SELECT name,submittime FROM enterprise WHERE 
YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now()); 

查询上周的数据 
SELECT name,submittime FROM enterprise WHERE 
YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now())-1;

 查询当前月份的数据 
select name,submittime from enterprise   where 
date_format(submittime,'%Y-%m')=date_format(now(),'%Y-%m') 查询距离当前现在6个月的数据 
select name,submittime from enterprise where submittime between date_sub(now(),interval 6 month) and now(); 

查询上个月的数据 
select name,submittime from enterprise   where 
date_format(submittime,'%Y-%m')=date_format(DATE_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m') 
select * from ` user ` where DATE_FORMAT(pudate, ' %Y%m ' ) = DATE_FORMAT(CURDATE(), ' %Y%m ' )  
select * from user where WEEKOFYEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = WEEKOFYEAR(now()) select *  from user  
where MONTH (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = MONTH (now()) select *  from [ user ]  
where YEAR (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = YEAR (now()) and MONTH (FROM_UNIXTIME(pudate, ' %y-%m-%d ' )) = MONTH (now())
```

### 参考链接


[mysql查询今天,昨天,近7天,近30天,本月,上一月数据方法 - 吕滔博客](http://www.lvtao.net/database/mysql-date-format.html)

[mysql数据库要按当天、昨天、前七日、近三十天、季度、年查询_百度文库](http://wenku.baidu.com/link?url=k_5IVPh5K5f57Locung0gPMF7UvYVfXjl99GXr-Lw6b1pYTzyf7FsHHjm__Y2DruM4JoaPlS2z4-nsx5A8wfVl2IaN2PcvN0bFB8xtQvkxq)


**清空表内数据** 

TRUNCATE TABLE teacher  其中teacher为表名

**数据库备份与恢复**  

**备份**

mysqldump -u 用户名 -p 数据库名 > 文件名.sql

**恢复**

mysqldump -u 用户名 -p 数据库名 > 文件名.sql

**注意**   

以上均在CMD中操作  

**实例**

```
mysqldump -u root -p test > test.sql

mysql -u root -p test < abc.sql
```


**表不存在则创建** 

```
创建数据库：

Create Database If Not Exists MyDB Character Set UTF8

创建数据表：

Create Table If Not Exists MyDB.MyTable(

ID Bigint(8) unsigned Primary key Auto_Increment,

thTime DateTime,

name  VarChar(128)

)Engine MyISAM
```