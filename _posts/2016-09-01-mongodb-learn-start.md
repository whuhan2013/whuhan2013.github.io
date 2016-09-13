---
layout: post
title: Mongodb数据库学习
date: 2016-9-1
categories: blog
tags: [数据库]
description: 数据库 
---

#### MongoDB简介                      
MongoDB(http://www.mongodb.org/)是一个高性能，开源(代震军大牛正在研究MongoDB的源码,大家可以去看看http://www.cnblogs.com/daizhj/)，模式自由(schema-free)的文档型数据库，它在许多场景下可用于替代传统的关系型数据库或键/值(key-value)存储方式。MongoDB使用C++开发，

具有以下特性：

- 面向集合的存储：适合存储对象及JSON形式的数据。
- 动态查询：MongoDB支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。
- 完整的索引支持：包括文档内嵌对象及数组。MongoDB的查询优化器会分析查询表达式，并生成一个高效的查询计划。
- 查询监视：MongoDB包含一个监视工具用于分析数据库操作的性能。
- 复制及自动故障转移：MongoDB数据库支持服务器之间的数据复制，支持主从模式及服务器之间的相互复制。复制的主要目标是提供冗余及自动故障转移。
- 高效的传统存储方式：支持二进制数据及大型对象（如照片或图片）。
- 自动分片以支持云级别的伸缩性（处于早期alpha阶段）：自动分片功能支持水平的数据库集群，可动态添加额外的机器。
模式自由（schema-free)，意味着对于存储在MongoDB数据库中的文件，我们不需要知道它的任何结构定义。如果需要的话，你完全可以把不同结构的文件存储在同一个数据库里。
- 支持Python，PHP，Ruby，Java，C，C#，Javascript，Perl及C++语言的驱动程序，社区中也提供了对Erlang及.NET等平台的驱动程序。

#### 使用场合:

- 网站数据：MongoDB非常适合实时的插入，更新与查询，并具备网站实时数据存储所需的复制及高度伸缩性。
缓存：由于性能很高，MongoDB也适合作为信息基础设施的缓存层。在系统重启之后，由MongoDB搭建的持久化缓存层可以避免下层的数据源过载。

- 大尺寸，低价值的数据：使用传统的关系型数据库存储一些数据时可能会比较昂贵，在此之前，很多时候程序员往往会选择传统的文件进行存储。

- 高伸缩性的场景：MongoDB非常适合由数十或数百台服务器组成的数据库。MongoDB的路线图中已经包含对MapReduce引擎的内置支持。

- 用于对象及JSON数据的存储：MongoDB的BSON数据格式非常适合文档化格式的存储及查询。
所谓“面向集合”（Collenction-Orented），意思是数据被分组存储在数据集中，被称为一个集合（Collenction)。每个集合在数据库中都有一个唯一的标识名，并且可以包含无限数目的文档。集合的概念类似关系型数据库（RDBMS）里的表（table），不同的是它不需要定义任何模式（schema)。


#### 安装MongoDB        

详情可参见官网：[Ubuntu下安装mongodb](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)


#### 常见用法 

**启动/停止/重启MongoDB**     

```
sudo service mongod start
sudo service mongod stop
sudo service mongod restart
查看状态
sudo service mongod status
```


**查看数据**    

```
MongoDb 命令查询所有数据库列表 

CODE: 

> show dbs 

如果想查看当前连接在哪个数据库下面，可以直接输入db 
CODE: 

> db 
Admin 
想切换到test数据库下面 
CODE: 

> use test 
switched to db test 
> db 
Test 
想查看test下有哪些表或者叫collection，可以输入 
CODE: 

> show collections 
system.indexes 
user 
想知道mongodb支持哪些命令，可以直接输入help 
CODE: 
> help 
```

**mongodb的基本操纵**

```
show dbs;// 查看当前系统有多少数据库
use qq    //使用use切换数据库
db.dropDatabase();//删除数据库
use qq //此处无需创建数据库，MONGOd会在需要的时候自己创建数据库
db.qq_data.insert({x:1}) //写入的数据为JSON数据
show dbs
//能看到qq数据库重新创建了
查询数据库
db.qq_data.find()  //_id 全局唯一的字段mongod自己创建的
db.qq_data.insert({x:2,_id:1}) //_id可以自己指定
find 支持limit,skip(跳过)数据

插入多条数据，使用语法
for(i=3;i<100;i++)db.qq_data.insert({x:i})  // 插入了97条数据
db.qq_data.find().cout()//算出有多少条数据
mongodb支持连续操作 db.qq_data.find().skip(3),limit(2).sort({x:1})


数据的更新
查找的记录的条件 和更新的数据

db.qq_data.update({x:1},{x:999})
此处有两个参数
第一个参数表示查找的地方，第二个表示要改的地方
db.qq_data.insert({x:100,y:100,z:100})
db.qq_data.update({z:100},{$set:{y:99}}) // 此处表示若更改则会把X,Z的值覆盖 使用set能够避免覆盖
更改一条不存在的数据的时候 默认插入到数据库中
db.qq_data.update({y:100},{y:999},true)  //此处增加TRUE 表示增加一天不存在的数据
//update更新多条数据
db.qq_data.insert({c:1})
db.qq_data.insert({c:1})
db.qq_data.insert({c:1})
db.qq_data.update({c:1},{$set:{c:2}},false,true)
//删除数据
db.qq_data.remove({x:3})
//删除数据表
db.qq_data.drop()
//查看索引
db.qq_data.getIndexes()
//简历索引
db.qq_data.ensureIndex({x:1})   x= 1表示正向排序，x=-1表示逆向排序
```


![](http://img.blog.csdn.net/20150328213231005?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFvXzYyNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**参考链接**      

[在Ubuntu 14.04 64bit上安装MongoDB并测试](http://blog.csdn.net/tao_627/article/details/44706345)      


[Ubuntu下Mongodb的配置和使用](http://my.oschina.net/kakoi/blog/515603)


**mongodb备份还原导出导入**

mongodump备份数据库

常用命令格

```
mongodump -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -o 文件存在路径  
```

- 如果没有用户谁，可以去掉-u和-p。
- 如果导出本机的数据库，可以去掉-h。
- 如果是默认端口，可以去掉--port。
- 如果想导出所有数据库，可以去掉-d。

mongorestore还原数据库

常用命令格式

```
mongorestore -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 --drop 文件存在路径  
```

--drop的意思是，先删除所有的记录，然后恢复。

恢复所有数据库到mongodb中

```
[root@localhost mongodb]# mongorestore /home/zhangy/mongodb/   #这里的路径是所有库的备份路径 
```


**将MongoDB的collection导出成CVS**

```
mongoexport -h localhost -d dbname -c collname -f field1,field2 --csv -o output.csv
```

**参考链接**

[mongodb 备份 还原 导出 导入](http://blog.51yip.com/nosql/1573.html)

[如何将MongoDB的collection导出成CVS](https://segmentfault.com/q/1010000000136706)

