---
layout: post
title: Django搭建简易博客
date: 2016-8-8
categories: blog
tags: [python]
description: Django
---


**Django简易博客，主要实现了以下功能**       

1. 连接数据库    
2. 创建超级用户与后台管理   
3. 利用django-admin-bootstrap美化界面    
4. template,view与动态URL   
5. 多说评论功能   
6. Markdown与代码高亮        
7. 归档，AboutME和标签分类     
8. 搜索与ReadMore  
9. RSS与分页     


需要添加的安装包   

- pip install PyMySQL      
- pip install bootstrap-admin     
- pip install markdown       

#### 要注意的一些问题     

- 模板的位置       
由于django的版本与系统等原因，template的位置写法有些不同，摸索出了一个有用的写法  

```
TEMPLATE_PATH = os.path.join(BASE_DIR, 'templates')
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [TEMPLATE_PATH],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.contrib.auth.context_processors.auth',
                'django.template.context_processors.debug',
                'django.template.context_processors.i18n',
                'django.template.context_processors.media',
                'django.template.context_processors.static',
                'django.template.context_processors.tz',
                'django.contrib.messages.context_processors.messages',
                'django.template.context_processors.request'
            ],
        },
    },
]
```

**参考链接**   
[Introduce  Django搭建简易博客教程](https://andrew-liu.gitbooks.io/django-blog/content/index.html)

**源码地址**     
[Django博客](https://github.com/whuhan2013/diango_blog)

**效果图**

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/p1.jpg)
