---
layout: post
title: Django环境配置
date: 2016-8-7
categories: blog
tags: [python]
description: Django
---


**Django安装**            

```
#安装最新版本的Django
$ pip install  django 
#或者指定安装版本
pip install -v django==1.7.1
```

- 项目创建       
$ django-admin startproject my_blog
- 建立Django app              
$ python manage.py startapp article
- 运行程序              
$ python manage.py runserver              


**pip安装python模块权限报错解决**                  
[windows下pip安装python模块时报错总结 - 温柔易淡 - 博客园](http://www.cnblogs.com/liaojiafa/p/5100550.html)          

**pycharm运行django程序**            
配置: runserver 0.0.0.0:8000        
参考：[pycharm上运行django服务器端、以及创建app方法](http://www.cnblogs.com/qinjiting/p/4678893.html)      



#### diango使用mysql数据库

**设置数据库**                  
Django项目建成后, 默认设置了使用SQLite数据库, 在my_blog/my_blog/setting.py中可以查看和修改数据库设置:   

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'csvt',
        'USER': 'root',
        'PASSWORD': 'root',
        'HOST': '',
        'PORT': '',

    }
}
```

同时要在__init__.py中写以下代码       

```
import pymysql
pymysql.install_as_MySQLdb()
```

**pymysql**的安装        

```
pip install PyMySQL
```

**创建models**            
在my_blog/article/models.py下编写如下程序:

```
from django.db import models

# Create your models here.
class Article(models.Model) :
    title = models.CharField(max_length = 100)  #博客题目
    category = models.CharField(max_length = 50, blank = True)  #博客标签
    date_time = models.DateTimeField(auto_now_add = True)  #博客日期
    content = models.TextField(blank = True, null = True)  #博客文章正文

    #python2使用__unicode__, python3使用__str__
    def __str__(self) :
        return self.title

    class Meta:  #按时间下降排序
        ordering = ['-date_time']
```

其中__str__(self) 函数Article对象要怎么表示自己, 一般系统默认使用<Article: Article object> 来表示对象, 通过这个函数可以告诉系统使用title字段来表示这个对象

- CharField 用于存储字符串, max_length设置最大长度
- TextField 用于存储大量文本
- DateTimeField 用于存储时间, auto_now_add设置True表示自动设置对象增加时间


**同步数据库**           

```
$ python manage.py makemigrations
$ python manage.py migrate
```


**Django Shell**          
现在我们进入Django中的交互式shell来进行数据库的增删改查等操作


```
$ python manage.py shell
Python 3.4.2 (v3.4.2:ab2c023a9432, Oct  5 2014, 20:42:22)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>>
这里进入Django的shell和python内置的shell是非常类似的
>>> from article.models import Article
>>> #create数据库增加操作
>>> Article.objects.create(title = 'Hello World', category = 'Python', content = '我们来做一个简单的数据库增加操作')
<Article: Article object>
>>> Article.objects.create(title = 'Django Blog学习', category = 'Python', content = 'Django简单博客教程')
<Article: Article object>

>>> #all和get的数据库查看操作
>>> Article.objects.all()  #查看全部对象, 返回一个列表, 无对象返回空list
[<Article: Article object>, <Article: Article object>]
>>> Article.objects.get(id = 1)  #返回符合条件的对象
<Article: Article object>

>>> #update数据库修改操作
>>> first = Article.objects.get(id = 1)  #获取id = 1的对象
>>> first.title
'Hello World'
>>> first.date_time
datetime.datetime(2014, 12, 26, 13, 56, 48, 727425, tzinfo=<UTC>)
>>> first.content
'我们来做一个简单的数据库增加操作'
>>> first.category
'Python'
>>> first.content = 'Hello World, How are you'
>>> first.content  #再次查看是否修改成功, 修改操作就是点语法
'Hello World, How are you'

>>> #delete数据库删除操作
>>> first.delete()
>>> Article.objects.all()  #此时可以看到只有一个对象了, 另一个对象已经被成功删除
[<Article: Article object>]  

Blog.objects.all()  # 选择全部对象
Blog.objects.filter(caption='blogname')  # 使用 filter() 按博客题目过滤
Blog.objects.filter(caption='blogname', id="1") # 也可以多个条件
#上面是精确匹配 也可以包含性查询
Blog.objects.filter(caption__contains='blogname')

Blog.objects.get(caption='blogname') # 获取单个对象 如果查询没有返回结果也会抛出异常

#数据排序
Blog.objects.order_by("caption")
Blog.objects.order_by("-caption")  # 倒序

#如果需要以多个字段为标准进行排序（第二个字段会在第一个字段的值相同的情况下被使用到），使用多个参数就可以了
Blog.objects.order_by("caption", "id")

#连锁查询
Blog.objects.filter(caption__contains='blogname').order_by("-id")

#限制返回的数据
Blog.objects.filter(caption__contains='blogname')[0]
Blog.objects.filter(caption__contains='blogname')[0:3]  # 可以进行类似于列表的操作
```






