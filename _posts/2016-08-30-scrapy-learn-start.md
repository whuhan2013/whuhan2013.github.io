---
layout: post
title: scrapy爬取豆瓣电影新片榜
date: 2016-8-30
categories: blog
tags: [网络爬虫]
description: 爬虫豆瓣 
---

#### scrapy
scrapy 是python家族中最负盛名的爬虫框架，其他比较好使的是 urllib,urllib2,requests,pyquery等，scrapy在 github 上有10455颗star，其热门程度可见一斑（django才15000多），另外，scrapy的操作很django有些很相似的地方，很方面有python的django经验的人上手。

scrapy分为以下几个部分：

![](http://7xlen8.com1.z0.glb.clouddn.com/scrapy_work.jpeg)

- 引擎(Scrapy): 用来处理整个系统的数据流处理, 触发事务(框架核心)

- 调度器(Scheduler): 用来接受引擎发过来的请求, 压入队列中, 并在引擎再次请求的时候返回. 可以想像成一个URL（抓取网页的网址或者说是链接）的优先队列, 由它来决定下一个要抓取的网址是什么, 同时去除重复的网址

- 下载器(Downloader): 用于下载网页内容, 并将网页内容返回给蜘蛛(Scrapy下载器是建立在twisted这个高效的异步模型上的)

- 爬虫(Spiders): 爬虫是主要干活的, 用于从特定的网页中提取自己需要的信息, 即所谓的实体(Item)。用户也可以从中提取出链接,让Scrapy继续抓取下一个页面

- 项目管道(Pipeline): 负责处理爬虫从网页中抽取的实体，主要的功能是持久化实体、验证实体的有效性、清除不需要的信息。当页面被爬虫解析后，将被发送到项目管道，并经过几个特定的次序处理数据。

- 下载器中间件(Downloader Middlewares): 位于Scrapy引擎和下载器之间的框架，主要是处理Scrapy引擎与下载器之间的请求及响应。

- 爬虫中间件(Spider Middlewares): 介于Scrapy引擎和爬虫之间的框架，主要工作是处理蜘蛛的响应输入和请求输出。

- 调度中间件(Scheduler Middewares): 介于Scrapy引擎和调度之间的中间件，从Scrapy引擎发送到调度的请求和响应。


**而scrapy的流程如图，并且可归纳如下：**

- 首先下载器下载request回执的html等的response

- 然后下载器传给爬虫解析

- 接着爬虫解析后交给调度器过滤，查重等等

- 最后交给管道，进行爬取数据的处理


#### 实战：用scrapy爬取豆瓣新片榜

首先下载安装scrapy：          

在python3与ubuntu中安装scrapy还是有些麻烦，需要一些依赖和条件，详情参见：[install scrapy](http://doc.scrapy.org/en/latest/intro/install.html)


然后启动一个爬虫：

```
scrapy startproject douban_new_movie
```

scrapy便会帮你初始化一个项目，项目文件包括：items.py（定制需要储存的文件的域，类似于orm）,pipelines.py（scrapy的定制管道）, settings.py（设置相关参数）和一个 spider文件夹（定制你的爬虫）

首先，编辑items.py 文件：


```
# -*- coding: utf-8 -*-


import scrapy


class DoubanNewMovieItem(scrapy.Item):

    movie_name=scrapy.Field()
    movie_star=scrapy.Field()
    movie_url=scrapy.Field()
```


- 首先引入scrapy

- 接着创建一个类，继承自scrapy.item,这个是用来储存要爬下来的数据的存放容器，类似orm的写法，

- 我们要记录的是，1.电影的名字，2.电影的评分，3.电影的链接

好，这个时候我们可以在 spider文件夹下创建一个 douban_new_movie_spider.py的文件，我们来编写我们的第一个爬虫，

在我们编写爬虫之前，先了解一下scrapy的爬取机制，scrapy和绝大多数的爬虫喜欢用繁琐的正则表达式不同，他更喜欢使用xpath和css的class来搜索他要的信息，笔者试过很多正则表达式的框架，首先正则表达式本来就是个很繁琐的东西，而且经常会出错，重写正则表达式就会耗去大量的开发时间，而xpath本就是为了解析html和xml而做的，非常方便，

不懂xpath的同学可以点这里： [xpath教程](http://www.w3school.com.cn/xpath/)

然后我们再来分析一下网页的源代码：

我们可以看到他们的xpath的组成，可以写出爬虫代码douban_new_movie_spider.py如下：

```
from scrapy.spiders import Spider
from scrapy.selector import Selector
from scrapy.http import Request
from douban_new_movie.items import DoubanNewMovieItem

class DoubanNewMovieSpider(Spider):
    name="douban_new_movie_spider"

    allowed_domains=["www.movie.douban.com"]

    # start_urls=[
    # 'http://movie.douban.com/chart'
    # ]

    headers = {
        "Accept": "*/*",
        "Accept-Encoding": "gzip,deflate",
        "Accept-Language": "en-US,en;q=0.8,zh-TW;q=0.6,zh;q=0.4",
        "Connection": "keep-alive",
        "Content-Type": " application/x-www-form-urlencoded; charset=UTF-8",
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.111 Safari/537.36",
        "Referer": "http://www.movie.douban.com/"
    }

    def start_requests(self):
        return [Request("http://movie.douban.com/chart", headers=self.headers)]

    def parse(self,response):
        sel=Selector(response)

        movie_name = sel.xpath("//a[@class='nbg']/@title").extract()
        movie_url=sel.xpath("//div[@class='pl2']/a/@href").extract()
        movie_star=sel.xpath("//div[@class='pl2']/div/span[@class='rating_nums']/text()").extract()


        item=DoubanNewMovieItem()

        item['movie_name']=[n for n in movie_name]
        item['movie_star']=[n for n in movie_star]
        item['movie_url']=[n for n in movie_url]

        yield item

        print(movie_name,movie_star,movie_url)
```

- 首先我们先从scrapy中获得所需要的通用的 spider和 selector

- 接着把我们的“储存容器”的items拿过来

- 创建一个爬虫，scrapy的爬虫必须要有以下几个参数：name（爬虫的名字，scrapy需要这个来找到所需要的爬虫），start-urls（这个是启动事的url，是一个python的list），parse（用来对response进行处理的方法）

- 可以看到我们的name叫做 douban_new_movie_spider，我们的start_urls是直接从这个页面开始的，

- 接着我们来看我们的parse方法，首先我们使用scrapy内置的selector搜索器，用搜索器的xpath进行搜索，

- 注意：seletor的方法返回后一定要用它的 extract()方法，来返回一个列表

- 接着，我们把得到的数据保存在我们的items容器中

- 最后，我们来返回我们的items，他会交给pipelines处理

好了，其实到现在，一个基础的爬虫就已经写好了，现在，我想把这些保存成json格式的数据。怎么做呢？当然是处理我们的数据啦，在哪里处理呢？当然是在我们的管道(pipelines)里处理啦，好我们来写一个处理程序—— pipelines.py：


```
# -*- coding: utf-8 -*-


import json
import codecs
import sys
import importlib

importlib.reload(sys)




class DoubanNewMoviePipeline(object):
    def __init__(self):
        self.file=codecs.open('douban_new_movie.json',mode='wb',encoding='utf-8')

    def process_item(self, item, spider):
        line='the new movie list:'+'\n'

        for i in range(len(item['movie_star'])):
            movie_name={'movie_name':str(item['movie_name'][i]).replace(' ','')}
            #print(movie_name)
            movie_star={'movie_star':item['movie_star'][i]}
            movie_url={'movie_url':item['movie_url'][i]}
            line=line+json.dumps(movie_name,ensure_ascii=False)
            line=line+json.dumps(movie_star,ensure_ascii=False)
            line=line+json.dumps(movie_url,ensure_ascii=False)+'\n'

        self.file.write(line)

    def close_spider(self,spider):
        self.file.close()
```

好，这个就是我们的管道程序：

- 首先，我们把我们的json包和codecs包引进，codecs包使用来处理中文的

- 接着，因为linux下对中文的支持的问题，如果你现在直接处理会报错，因为linux操作系统对中文的储存问题，而在windows和mac下则没问题

- 好的，我们先打开一个 douban_new_movie.json文件，我们将把数据储存在这个文件内，注意他的编码

- scrapy的pipeline一般包括startspider,proessitem,closespider方法，其中processitem最为重要，我的程序写的很清楚啦，唯一要提醒的是，这个是由spider返回的item来处理，你可以向处理字典一样处理他们

- 这里，因为电影名，电影评分和电影链接是一一对应的，所以，我直接使用了电影列表的长度来调控他们

- 接着你需要把一些东西写进settings.py来告诉scrapy你将用什么pipeline：

在settings.py后面加上一句：

```
ITEM_PIPELINES={
    'douban_new_movie.pipelines.DoubanNewMoviePipeline':300,
}
```

好，我们的爬虫就算完成了，我们来启动他，见证奇迹的时刻吧！！

在终端输入，就可以进行爬取：

```
scrapy crawl douban_new_movie_spider
```

你就会发现同文件夹下出现了一个新文件 douban_new_movie.json

![](http://7xlen8.com1.z0.glb.clouddn.com/scrapy_result.png)


**源码地址**           

[pythoncode/douban_new_movie at master](https://github.com/whuhan2013/pythoncode/tree/master/douban_new_movie)

**参考链接**                

[用scrapy爬取豆瓣电影新片榜](http://aljun.me/post/4)