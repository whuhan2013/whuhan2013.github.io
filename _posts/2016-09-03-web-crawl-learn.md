---
layout: post
title: 爬虫基础知识
date: 2016-9-2
categories: blog
tags: [网络爬虫]
description: 爬虫58 
---

### 爬虫介绍             
网络爬虫，英译为 web crawler ，是一种自动化程序，现在我们很幸运，生处互联网时代，有大量的信息在网络上都可以查得到，但是有时我们需要网络上的数据，活着文章，图片等等，但是，一个个地复制，粘贴是不是太傻了，循着 “DRY” 的设计原则，我们希望用一个自动化的程序，自动帮我们匹配到网络上面的数据，然后下载下来，为我们所用。

其中，搜索引擎就是个很好的例子，搜索引擎技术里面大量使用爬虫，他爬取下整个互联网的内容，存储在数据库里面，做索引。

#### 爬虫思路            
首先，我们要知道，每一个网页都是一份 HTML文档，全称叫 hypertext markup language，是一种文本标记语言，他长的就像这样：

```
<html>
    <head>
        <title>首页</title>
    </head>
    <body>
        <h1>我是标题</h1>
        <img src="xxx">
    </body>
</html>
```


由上，我们可以看出，这是一份很有规则的文档写法

我们打开一个网页，即是通过了 HTTP协议，对一个资源进行了请求，如何他返还你一份 HTML文档，然后浏览器进行文档的渲染，这样，你就看到一份美丽的网页啦

所以，我们只需要模拟浏览器，发送一份请求，获得这份文档，再抽取出我们需要的内容就好

#### 简单爬虫                                       
我们使用python语言，因为python语言的网络库非常多，而且社区对于爬虫的建设非常好，现在很多情况下，大家说起爬虫，第一个想到的就是python，而且，当年GOOGLE的部分爬虫也是使用python编的，只不过后面转去了C++，这也说么python对爬虫是得天独厚的

那么，我们来写一个最简单的爬虫：

```
import urllib2

response=urllib.urlopen("http://xxx.com")
```


首先，我们引入python的urllib库
然后，我们对一个url地址进行反问
这样，只要我们运行：

```
print response.read() 
```

我们就可以看到，一版面的 HTML代码了，就是这么简单，使用python

#### 进阶1                       
对于一般网站，其实，刚刚那样的程序就够用了，但是，网页里面的数据对于很对互联网企业也很宝贵，所以，她们并不想让你随意拿走，他要做一些手脚，防止你爬取

对于这重情况，我们的唯一办法就是让自己装的更像浏览器

首先，我们来看看一个简单的请求

![](http://7xlen8.com1.z0.glb.clouddn.com/C5C103CB-1B64-4D7C-A68F-309FF44DDF49.png)

可以看见，我们的一个访问，包含了很多信息，通常我们必须要伪装的就是 User-Agent了

我们的手法是这样的：

```
import urllib2

header={
    "User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:43.0) Gecko/20100101 Firefox/43.0",
    "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
    "Host":"aljun.me"
}
request=urllib2.request("http://xxx.com",header=header)
response=urllib2.urlopen(request)
```

我们先伪造了我们请求头请求，然后再发送我们的请求，这样做的就好像真的是浏览器发送的一样
但是啊，有时候，我们会遇到一个东西，叫做cookie，如果你熟悉互联网发展时，就会知道，这个是网景公司推出的一种想法，他能在用户浏览器用存储信息，这样就做到了登录主场购物车等等操作的浏览器回话，就能够确认访问者的身份

那么我们要怎么样做，才能把我们的cookie发送过去呢？

```
import urllib2
import cookielib

cookie={"bdshare_firstime":1455378744638}
cookie = cookielib.CookieJar()

opener = urllib2.build_opener(urllib2.HTTPCookieProcessor())

urllib2.install_opener(opener)

response=urllib2.urlopen("http://xxx.com")
```


这样我们就构造了一个cookie发送过去，不过我最近遇到需要cookie的情况比较少，而且真正需要cookie的时候，其实交互还是很多的，单一的构造cookie请求我感觉是不够的

现在我们可以发送cookie了，那么又迎来了另一个问题，我们一直在使用没有参数的访问方法，想象一下，以往我们访问网页，有时是需要输入几个登陆框，或者评论框的，这样我们才能有数据交互

其实这个也很简单，我们可以这么写：

```
import urllib2

data={
    "username":"xxx"
    "password":"xxx"
}
request=urllib2.request("http://xxx.com",data)
response=urllib2.urlopen(request)
```

这样做便可以了，那么组合以上我们的操作，我们就基本可以访问一切我们想要访问的网页，获得一份 HTML文档了


### 爬取图片   
突然发现，若只是获得文本类型的HTML文档，好像很无聊，那么我们应该怎么样下载到我们的图片呢？

首先，你要知道，图片是有地址的，比如

http://zhaduixueshe.com/static/pic/discovery.png              
可以看出，他也是一个文件类型，只要对他进行HTTP访问，我们一样能拿到数据，一样可以下载下来

比较暴力的办法：

```
import urllib2

response=urllib2.urlopen("http://zhaduixueshe.com/static/pic/discovery.png")

with open("xxx.png","wb") as f:
    f.write(response.read())
```


这样确实可以成功的下载图片，但是比较暴力

那么我们可以温柔一点，因为有时图片较大，这样下载很容易出现错误，这个时候我们需要缓存的帮助

```
import urllib2
import stringIO

response=urllib.urlopen("http://zhaduixueshe.com/static/pic/discovery.png")

response=stringIO.stringIO(response.read())

with open("xxx.png","wb") as f:
    f.write(response)
```

而且这样的做法，还可以使用PIL模块，对下下来的图片进行渲染活着整理

那么，最好用的下载图片的办法是什么呢？

```
import urllib

path="xxx.png"
url="http://zhaduixueshe.com/static/pic/discovery.png"

urllib.urlretrieve(url,path)
```

这是官方推荐做法，非常快，而且好用。


### requests            
这个世界上，总有那么一些人，他们不满现状，积极进取，python内置的urllib和urllib2其实已经算是蛮好用了，但是非有人不服，于是他做出了更好的一个http库，叫做request

Requests: HTTP for Humans
它是以这么一句话介绍自己的，为人类使用的HTTP库

下载requests：

```
pip install requests
```

他的使用非常之简单，简直可以说是弱智

接下来我演示部分他的使用，光是看语句，你便知道他的作用都是什么

```
In [1]: import requests

In [3]: response=requests.get("http://aljun.me")

In [4]: response=requests.post("http://zhihu.com/login",data={"username":"xxx"})
```

等等之类的操作，由于他的文档写的非常好，我还是比较推荐大家去看看他的官方文档

[requests官方文档](http://www.python-requests.org/en/master/user/quickstart/)


### 都爬下来了，然后呢？            
通常，我们对于HTML文档的处理的办法，比较流行的集中：

- re(正则表达式）
- beautifulsoup
- xpath
- pyquery
- re

首先，正则表达式是什么呢？它是用来匹配文档的，例如

```
import urllib2
import re

reg=r'http.(d+).jpg'
reg=re.compile(reg)
response=urllib2.urlopen("http://xxx.com")
result=re.findall(response.read(),reg)
```

这样我们就得到了我们想要的，符合我们需要的信息了

但是古话有云：

你遇到了一个问题，你想到使用正则表达式解决它，于是，你现在有了两个问题
即是说，正则这个东西很厉害，但是不是很好掌握，反正我是从来没背下来几个正则表达式匹配模式的

**beautifulsoup**

这个库是用来编译HTML代码的专业库

```
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc, 'html.parser')

print(soup.prettify())

###
<html>
    <head>
        <title>首页</title>
    </head>
    <body>
        <h1>我是标题</h1>
        <img src="xxx">
    </body>
</html>

soup.title
# <title>The Dormouse's story</title>

soup.title.name
# u'title'

soup.title.string
# u'The Dormouse's story'

soup.title.parent.name
# u'head'

soup.p
# <p class="title"><b>The Dormouse's story</b></p>

soup.p['class']
# u'title'

soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find(id="link3")
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
```

是不是有一种要什么有什么的感觉

之后的xpath，我在之前的教程里面有介绍过了，

**pyquery**

pyquery是以jquery的选择器语法为基础，非常适合前端转来的

```
>>> from pyquery import PyQuery as pq
>>> from lxml import etree
>>> import urllib
>>> d = pq("<html></html>")
>>> d = pq(etree.fromstring("<html></html>"))
    >>> d = pq(url=your_url)
>>> d = pq(url=your_url,
...        opener=lambda url, **kw: urlopen(url).read())
>>> d = pq(filename=path_to_html_file)

>>> d("#hello")
[<p#hello.hello>]
>>> p = d("#hello")
>>> print(p.html())
Hello world !
>>> p.html("you know <a href='http://python.org/'>Python</a> rocks")
[<p#hello.hello>]
>>> print(p.html())
you know <a href="http://python.org/">Python</a> rocks
>>> print(p.text())
you know Python rocks
```

**调用json格式**     

前面我们说到了网络资源不仅仅是HTML，还有一种格式叫json文件

它是javascript面向对象表达式的意思

很多网站都会提供api，也就是数据接口，一般都是以json格式返回的，我的博客的like返回也是json格式的文件

```
In [1]: import urllib2

In [2]: response=urllib2.urlopen("http://aljun.me/like")

In [3]: print response.read()
{
  "liked": 1647
}
```

我可以良好的使用json包，来对这个文件进行解析，因为json格式和python自带的dict文件形式很像，所以，这个非常简单


### 性能进阶           

一般的进阶，从性能上来看，一般是这么几个模块：

- threading
- multiprocessing
- twisted
- gevent
- tornado


#### threading

首先，第一个，由于python内部又 GIL 锁这种东西的存在，python的多线程编程其实并不算十分友好的，python想保证他的线程安全，线程内共享资源共享很多东西，而且更好的做通信，但是最近和几个大牛谈过之后，他们都说python多线程不好，我也就没多试，这里给出一个样板吧

这里给出一个多线程的爬虫例子：

```
import urllib2
import time
import Queue
import threading

hosts=["http://baidu.com",
   "http://jianshu.com",
   "http://taobao.com",
   "http://tmall.com",
   "http://jd.com"]

queue=Queue.Queue()

class ThreadUrl(threading.Thread):
    def __init__(self,queue):
        threading.Thread.__init__(self)
        self.queue=queue

    def run(self):
        while True:
            host=self.queue.get()
            url=urllib2.urlopen(host)
            print url.geturl()
            print self.getName()

            self.queue.task_done()


start=time.time()

def main():
    for i in range(5):
        t=ThreadUrl(queue)
        t.setDaemon(True)
        t.start()

    for host in hosts:
        queue.put(host)
    queue.join()

if __name__=="__main__":
    main()

print "it use time:%s" % (time.time()-start)
```

这里请一定要用队列queue来进行任务队列的设置,然后最好设置成 setDaemon(True)这样可以保护线程       


#### multiprocessing

你要充分利用好自己的cpu资源，那么肯定是要选择多进程的，多进程的程序问题在资源的通信上面，换在爬虫上，就是任务队列的设置，通常，放在生产上面，我们会使用 redis或者 celery这样的来做我们的任务队列

那么python对于多进程支持算是还不错的，通常你复制出一个子进程，是这么写：

```
import os
pid2=os.fork()
print str(os.getpid)
print str(os.getppid)
```


这样可以获得一个子进程，然后这个子进程的资源是完全复制父进程的，你这样写可以获得父进程的pid号码

python的multiprocessing模块，方便我们进行多进程编写，下面是一个多进程爬虫的例子：

```
from multiprocessing.dummy import Pool
import requests
import urllib
import time
import os

def get_source(url):
    print "we crawled:%s" % url
    html=urllib.urlopen(url)
    print html.geturl()
    print os.getpid()

urls=[]

for i in range(1,21):
    new_page="http://tieba.baidu.com/p/3522395718?pn="+str(i)
    urls.append(new_page)

time1=time.time()

for url in urls:

    get_source(url)

time2=time.time()
print "it take:",(time2-time1)

pool=Pool(4)
time3=time.time()
res=pool.map(get_source,urls)
pool.close()
pool.join()
time4=time.time()
print "now,it take,",(time4-time3)
```

这里的pool是启动一个进程池，一般是你有几核的CPU你有写几个，你通过调pool.map对你的队列进行任务映射
最后时一定记得要pool.close之后才 pool.join这样最后关掉你的进程


#### gevent

首先我们先来讲讲什么的 协程

协程的概念很早就提出来了，但直到最近几年才在某些语言（如Lua）中得到广泛应用。

子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。

所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。

子程序调用总是一个入口，一次返回，调用顺序是明确的。而协程的调用和子程序不同。

协程看上去也是子程序，但执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。

这样可以实现大并发的执行，但是却又不消耗太多资源

python对协程的支持，是以迭代器来支持的，通常是使用 yield表达协程

这里我们使用 gevent，它内部基于greenlet的机制，非常敏捷

gevent我早早便在我的django的web应用里面，让他配合gunicorn一起使用，并发的性能比起uwsgi好了一个级别

那么我们来看看普通协程的程序怎么写：


```
import gevent.monkey
gevent.monkey.patch_socket()

import gevent
import urllib2

def fetch(pid):
   response = urllib2.urlopen('http://aljun.me')
   result = response.read()


   print('Process %s: %s' % (pid, datetime))


def synchronous():
    for i in range(1,10):
        fetch(i)

def asynchronous():
    threads = []
    for i in range(1,10):
        threads.append(gevent.spawn(fetch, i))
    gevent.joinall(threads)

print('Synchronous:')
synchronous()

print('Asynchronous:')
asynchronous()
```

对比同步异步，时间上快乐相当多

这里使用非常简单，我们只需把我们的function 放进 gevent.spawn 里面
然后，将这些，传给 gevent.joinall便可以了

同样的，我们也可以使用诸如：

```
from gevent.pool import Pool

p = Pool(4)

p.map()
```

就和multiprocessing一样的操作。

同时，gevent模块的monkey_patch可以将你的socket类型修改，提升性能，因为我对这个研究不算多，暂且不说


#### 异步

首先，我们先谈谈什么是异步：             
而twisted是基于一个react的异步模型，scrapy就是基于他做的，我们已经见识过scrapy的威力。，因为twisted，人如其名，文档也很扭曲，看的十分不易：   

![](http://7xlen8.com1.z0.glb.clouddn.com/reactor-doread.png)

这个是twisted的模型，

下面我给一个基于twisted的爬虫例子把：

```
from twisted.internet import defer,reactor,task
from twisted.web.client import getPage

maxRun=2

urls=[
    'http://zhaiduixueshe.com',
    'http://baidu.com',
    'http://taobao.com',
]

def pageCallback(res):
    print len(res)
    return res

def doWork():
    for url in urls:
        d=getPage(url)
       d.addCallback(pageCallback)
       yield d

def finish(ign):
    reactor.stop()

def test():
    defereds=[]
    coop=task.Cooperator()
    work=doWork()
    for i in xrange(maxRun):
        d=coop.coiterate(work)
       defereds.append(d)

    dl=defer.DeferredList(defereds)
    dl.addCallback(finish)

test()
reactor.run()
```

而tornado的tornado.web.AsynHTTPClient算是一种服务，他基本算是HTTP库，所以先不讨论

综上的话，我是推荐：

- multiprocessing
- gevent     

这两个组合，你在使用mongodb做数据存取，用redis做任务的url缓存,用celery做调度队列，那么跑一个百万级别的爬虫还说相当简单的


**参考链接**    

[爬虫教程（1）基础入门](http://aljun.me/post/17)

[爬虫教程（2）性能进阶](http://aljun.me/post/18)