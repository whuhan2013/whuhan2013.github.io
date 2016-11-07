---
layout: post
title: 番剧推荐系统实现
date: 2016-11-7
categories: blog
tags: [python]
description: 番剧推荐系统实现
---

**本文主要包括以下内容**        

- Python 语言基础
- SQL 语言基础
- HTML 与 CSS 基础
- 数据库表的拆分与设计
- Flask 框架的使用
- MySQL 的 Python 接口的使用
- 复杂查询语句的使用
- 推荐算法的简易设计       


**mysql下载安装**       
[在Mac如何启动MySQL](http://jingyan.baidu.com/article/48a42057e2b2b9a9242504a2.html)       

安装mysqldb：sudo pip3 install PyMySQL      

**flask测试**       

```
from flask import Flask
app=Flask(__name__)
@app.route('/')
def index():
    return "hello"
if __name__=="__main__":
    app.run(debug=True)
```

运行编写的代码,使用浏览器访问 localhost:5000 查看 Flask Web 服务是否已经启动：      
![](https://dn-anything-about-doc.qbox.me/document-uid208579labid2090timestamp1472456816185.png/wm)     

**mysql测试**        
用pymysql代替mysqldb           

```
import pymysql
pymysql.install_as_MySQLdb()
db=pymysql.connect("localhost","root","root","recommend")
cursor=db.cursor()
sql="create table user_anime(user int,anime int)"
cursor.execute(sql)
db.close()
```

创建表成功则成功       


**创建数据表**   

- 创建 user 表存储 id，姓名，用 id 为主键，即 id 不能重复。
- 创建 anime 表存储 id，名称，介绍信息，同样用 id 为主键，即 id 不能重复。
- 创建anime_style表用于存储类型      
- 创建user_anime联系表       


#### 简单推荐算法实现

本节实验中我们实现的推荐算法比较简单，基本思路：

- 找到用户所喜爱的番剧
- 分析这些番剧的类别（一个番剧可能有多个标签），进行统计排序
- 找到前三个标签，从数据库中找到同时具有这三个标签的番剧（喜欢的不能再推荐）
- 将番剧相关信息（name，brief）进行展示     

SQL 数据库操作的实现上述的思路1 和 思路2：

下面代码实现得到用户所喜欢top3类型

```
    sql='''
    select style_id   from
        (select user_id,style_id from 
        (select user_id,anime_id as id from user_anime where user_id=%s) as s 
        natural join anime natural join 
        (select anime_id as id,style_id from anime_style) as n
         )as temp group by style_id order by count(user_id) desc limit 3;'''%user
```

其中下图所划去的一行内容对应思路1，即从 user_anime 表中查询用户和喜欢的番剧数据对：

```
(select user_id,anime_id as id from user_anime where user_id=%s) as s
```


完整的recommend.py代码     

```
# -*- coding: utf-8 -*-
"""
Created on Thu Aug 25 18:47:40 2016

@author: Albert
"""

from random import choice
import MySQLdb

def recommend(user):
    #连接数据库
    DB=MySQLdb.connect("localhost","root","","recommend")
    #获得数据库游标
    c=DB.cursor()

    #下面代码为实现从数据库中得到用户user所喜欢的番剧编号，以便判断重复
    love=[]
    #sql语句
    sql="select anime_id  from user_anime where user_id=%s"%user
    c.execute(sql)
    #得到结果集
    results=c.fetchall()
    for line in results:
        love.append(line[0])

    #下面代码实现得到用户所喜欢top3类型
    sql='''
    select style_id   from
        (select user_id,style_id from 
        (select user_id,anime_id as id from user_anime where user_id=%s) as s 
        natural join anime natural join 
        (select anime_id as id,style_id from anime_style) as n
         )as temp group by style_id order by count(user_id) desc limit 3;'''%user

    c.execute(sql)
    results=c.fetchall()
    lis=[]
    anime={}
    for (line,) in results:

        lis.append(line)
    for i in lis:
        #从番剧信息的数据库中得到Top3各个类别的所有番剧并存入anime字典中
        sql="select anime_id from anime_style where style_id="+str(i)+";"
        c.execute(sql)
        results=c.fetchall()  
        anime_lis=[]
        for result in results:
            anime_lis.append(result[0])
        #类型为key，值为存放番剧数据的列表
        anime[str(i)]=anime_lis 
    #建立三个类别番剧的set（集合），并取交集，即得到同时有三个类型标签的番剧
    s=set(anime[str(lis[0])])&set(anime[str(lis[1])])&set(anime[str(lis[2])])

    #建立用户喜欢番剧的集合
    loveSet=set(love)
    #如果系统得到的所有番剧用户均已看过了即loveSet>s，就从Top1类型即最喜欢的类型中挑选一个
    #这里我们默认数据库量够大，即用户没有把一个类别的所有番剧看完
    if loveSet>s:
        s=set(anime[str(lis[0])])

    #把集合转化为列表以待随机函数的使用
    set_lis=[]
    for i in s:
        set_lis.append(i)
    #从结果中随机挑选
    recommend=choice(set_lis) 
    #直至挑选到用户没看过的
    while  recommend in love:
        recommend=choice(set_lis)
    dic={}
    #得到推荐番剧的相关信息，返回以待使用，显示
    sql="select name,brief from anime where id="+str(recommend)+";"
    c.execute(sql)
    results=c.fetchall()
    dic['name']=results[0][0]
    dic['brief']=results[0][1]

    DB.close()
    return dic
```

#### 推荐系统的实现及部署

app.py 的实现

```
# -*- coding: utf-8 -*-
"""
Created on Mon Aug 29 19:45:40 2016

@author: Albert
"""

from flask import Flask,render_template,request
import recommend
app=Flask(__name__)
@app.route('/')
def index():
    return render_template('index.html')
@app.route('/search/')
def search():
    #request为全局变量可得到用户输入信息
    n=request.args.get('user')
    #调用推荐函数
    dic=recommend.recommend(n)
    #用返回结果进行模板渲染
    return render_template('search.html',Data=dic)
if __name__=="__main__":
    app.run(debug=True)
```
模板文件必须放在templates文件夹下      

![](https://dn-anything-about-doc.qbox.me/document-uid208579labid2091timestamp1472457927044.png/wm)       
![](https://dn-anything-about-doc.qbox.me/document-uid208579labid2091timestamp1472457927211.png/wm)


*参考：**[推荐系统的实现](https://www.shiyanlou.com/courses/633/labs/2091/document)         
**SourceCode**[Suggest](https://github.com/whuhan2013/pythoncode/tree/master/suggest)
















