---
layout: post
title: 豆瓣top250电影数据分析
date: 2016-9-1
categories: blog
tags: [网络爬虫]
description: 爬虫豆瓣 
---

**概述**        

1. 使用scrapy爬虫获得top250数据       
2. 使用mongodb存储数据 
3. 使用matplotlib进行数据可视化


#### 使用scrapy爬虫获得top250数据               

```
from scrapy.spiders import Spider
from scrapy.linkextractors import LinkExtractor
from scrapy.selector import Selector
from douban_movie_category.items import DoubanMovieCategoryItem
from scrapy.spiders import Rule,CrawlSpider
import jieba
class CategorySpider(CrawlSpider):
    name = "category_spider"
    download_delay = 1

    allowed_domains = []

    start_urls = [
        'http://movie.douban.com/top250?start=0&filter=',
    ]

    rules = (
        Rule(LinkExtractor(allow=(r'\?start=\d+&filter=')),
             callback='parse_item'),)

    def parse_item(self, response):
        sel = Selector(response)
        item = DoubanMovieCategoryItem()
        category = sel.xpath("//div[@class='info']/div[@class='bd']/p/text()").extract()

        print(type(category))
        x = []
        for i in category:

            if len(i) > 5 and ':' not in i:
                i = i.split('/')
                i = i[len(i) - 1]

                i = i.strip()
                i = i.replace(" ", "")
                word = i

                if word != " " and len(word) > 0:

                    print(len(word))
                    print(word)

                    words = jieba.cut(word, cut_all=False)
                    for n in words:
                        print(n)
                        x.append(n)
        item['categories'] = x
        yield item

```

这里我们对他进行了稍许的字符串处理，因为过程很麻烦，涉及到了中文的encoding编码之类的问题，花了我很长时间，而且这段逻辑代码看着很丑陋，我就简单的说说要点吧

- 首先下载下来的是这个带着 “导演：```````犯罪 剧情”的这么一段长长的文字，我们对他进行去除空格，并且使用 / `来裂开他们，然后我们发现裂开后的最后一个元素就是我们想要的，这是我们把它取出来

- 第二点，这就要感谢我们天朝人们出品的动用很多机器学习算法的包 jieba，通过他的只能分词，我们可以把诸如 犯罪剧情这样的词组分成 [ u"犯罪", u"剧情"]这样，我们就能得到这个页面的所有出现过的分类了

好了，现在我们来编写我们的 pipelines.py，这次我们要使用 mongodb来保存我们爬取的数据，并且给予它一定的数数，因为我长期在写后台时基于的是 sql型的数据库，这次使用 pymongo调用使用 mongodb 感觉还不错，数度相当快，而且也非常好操作，返回回来的字典与他本身保存的 json格式很好相通

**注意**       
由于豆瓣有一些反爬操作，所以需要在setting.py中设置   

```
#设置代理，有些网站是反爬虫，所以要将其伪装成浏览器
USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/536.5 (KHTML, like Gecko) Chrome/19.0.1084.54 Safari/536.5'
```

#### 使用mongodb存储数据      

```
import pymongo
import sys

class DoubanMovieCategoryPipeline(object):
    def __init__(self):
        self.mongo_url = 'localhost'
        self.mongo_port = 27017
        self.collection_name = 'categories'
        self.db_name = 'douban_category'

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_url, self.mongo_port)
        self.db = self.client[self.db_name]
        self.con = self.db[self.collection_name]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        for i in item['categories']:
            if self.con.find_one({"category": i}):
                add_one = self.con.find_one({'category': str(i)})
                print(add_one)
                count = int(add_one['count'])
                count = count + 1
                add_one['count'] = count
                print(add_one)
                print("here1")
                print(count)
                self.con.save(add_one)
            else:
                self.con.insert_one({'category': i, "count": 1})
                print("here2")
                print("hello")
        print("here3")
        print(item['categories'])
```

- 这里我们每次爬虫都开启我们的 数据库，进行数据库操作，每次关掉爬虫就关掉（这点比去 sql型数据库的 orm舒服多了）

- 我们做一次我们得到的数据的迭代处理，若是数据库里面有的我们就在这里加1，若是没有我们就创建，初始出现次数 count为1

好的，那么将 pipelines在 settings.py中设定好，就和我前几次一样

我们就可以跑动我们的代码：

```
scrapy crawl category_spider
```

如此一来，我们变得到了他的数据


#### matplotlib数据可视化       

我们这次使用的是 python家族做数据可视化久负盛名的包 matplotlib，他里面包含了大量的可视化操作需要调用的函数，可以画做诸如 饼状图，散点图，函数曲线图，柱状图，气象图，3d渲染图等等等等 ，因为他和 巨强大的numpy与 我还没有使用过的 scipy python家族中做科学计算的必备，也是因为有了他们，python得以在科学计算领域与 matlab 相抗衡。我们还是老规矩吧，先上代码再进行他的思路解释：


```
import matplotlib.pyplot as plt
from pylab import *

import pymongo
from matplotlib.font_manager import FontProperties
import subprocess
import numpy as np


font=FontProperties(fname=r'/usr/share/fonts/truetype/字体管家扁黑体.ttf',size=14)



def pic_show():
    client=pymongo.MongoClient("localhost",27017)

    db=client['douban_category']

    con=db['categories']

    data=[]
    category=[]

    for item in con.find():
        data.append(item['count'])
        category.append(item['category'])

    print(data)
    for n in  category:
        print(n)

    y_pos=range(len(category))

    colors = np.random.rand(len(category))

    plt.barh(y_pos,data,align='center',alpha=0.4)
    plt.yticks(y_pos,category,fontproperties=font)
    for data,y_pos in zip(data,y_pos):
        plt.text(data,y_pos,data,horizontalalignment='center',verticalalignment='center', weight='bold')
    plt.ylim(+28.0,-1.0)
    plt,title(u"豆瓣电影top250部分类统计",fontproperties=font)
    plt.ylabel(u"电影分类",fontproperties=font)
    plt.subplots_adjust(bottom = 0.15)
    plt.xlabel(u"分类出现次数（一部影片分类可以多个）",fontproperties=font)
    plt.savefig("douban_category_new.png")

pic_show()
```

- 首先，我们用 pymongo，把数据从数据库中拿出来，并按照 category（分类名），data（出现次数），来存储数据，因为他是和python的dict形式很像的，所以我们使用迭代把他们拿出来

- 然后我调用 matplotlib进行绘图，这次我们用的是柱状图(barh)，还需要在y轴上做分类名的标记(ystick)，并且打出他的标题(title)以及他在x轴边上是什么(xlabel)，y轴边上是什么(ylabel)，y轴的大小和可视面积(xlim)，最后我们把它保存成图片。

- 注意，这里要注意，                    matplotlib并没有包含中文字体，所以如果你想打中文上图片，便会出现方块而不是我们要的文字，所以你需要调用本地的 tff字体文件，让他打出中文！        


**源码地址：**[豆瓣TOP250数据分析](https://github.com/whuhan2013/pythoncode/tree/master/douban_movie_category)

**参考链接**      

[爬取豆瓣top250电影榜的电影分类进行数据分析](http://aljun.me/post/9)

**结果如下**     

![](https://github.com/whuhan2013/pythoncode/blob/master/douban_movie_category/douban_movie_category/douban_category_new.png?raw=true)