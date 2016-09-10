---
layout: post
title: python刷CSDN访问量与IP代理 
date: 2016-9-10
categories: blog
tags: [网络爬虫]
description: python刷CSDN访问量与IP代理
---

使用了python3的urllib模块，开始使用了简单的urllib.request.urlopen()函数，结果发现行不通，csdn设置了简单的检查选项，需要python模拟浏览器进行访问才行，简单的很，那就模仿一个好啦，使用urllib.request.build_opener()就可以进行模拟啦，添加一个访问头就可以啦
 
但是呢，访问太频繁有可能会造成服务器拒绝访问，那么就稍微等等好啦，使用time模块中的sleep()函数即可。还有一个小问题，当服务器拒绝的时候，python会当成错误，从而终止了程序，这样就不好玩了，一点都不自动化，解决这个问题也蛮简单，刚才看书才看到try…except…语法，这样把出现的错误都放到except语句里面不就OK了么，经过本人测试，一般会出现下面两个错误urllib.error.HTTPError和urllib.error.URLError，那这两个错误都弄到except里面就可以啦


```
__author__ = 'Ricardo'

import urllib.request
import re
import time
from bs4 import BeautifulSoup

p = re.compile('/whuhan2013/article/details/........')

# 自己的博客主页
url = "http://blog.csdn.net/whuhan2013"

# 使用build_opener()是为了让python程序模仿浏览器进行访问
opener = urllib.request.build_opener()
opener.addheaders = [('User-agent', 'Mozilla/5.0')]

html = opener.open(url).read().decode('utf-8')

allfinds = p.findall(html)
# print(allfinds)

urlBase = "http://blog.csdn.net"  # 需要将网址合并的部分
# 页面中的网址有重复的，需要使用set进行去重复
mypages = list(set(allfinds))
for i in range(len(mypages)):
    mypages[i] = urlBase + mypages[i]

print('要刷的网页有：')
for index, page in enumerate(mypages):
    print(str(index), page)

# 设置每个网页要刷的次数
brushNum = 200

# 所有的页面都刷
print('下面开始刷了哦：')
for index, page in enumerate(mypages):
    for j in range(brushNum):
        try:
            pageContent = opener.open(page).read().decode('utf-8')
            # 使用BeautifulSoup解析每篇博客的标题
            soup = BeautifulSoup(pageContent)
            blogTitle = str(soup.title.string)
            blogTitle = blogTitle[0:blogTitle.find('-')]
            print(str(j), blogTitle)

        except urllib.error.HTTPError:
            print('urllib.error.HTTPError')
            time.sleep(3)  # 出现错误，停几秒先

        except urllib.error.URLError:
            print('urllib.error.URLError')
            time.sleep(3)  # 出现错误，停几秒先
        time.sleep(0.5)  # 正常停顿，以免服务器拒绝访问
```

参考链接：[Python 刷网页访问量](http://blog.csdn.net/calling_wisdom/article/details/40921021#)


但这样做存在一些问题，可能是因为IP没有发生变化，导致访问量上升一些以后又无效了，所以通过抓取一些代理IP来访问


```


# coding:utf-8

from gevent import monkey
monkey.patch_all()

import urllib.request
from gevent.pool import Pool


import requests
from bs4 import BeautifulSoup


class SpiderProxy(object):
    """黄哥Python培训 黄哥所写
    从http://www.xicidaili.com/wt 取代理ip
    """
    headers = {
        "Host": "www.xicidaili.com",
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:47.0) Gecko/20100101 Firefox/47.0",
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Accept-Language": "en-US,en;q=0.5",
        "Accept-Encoding": "gzip, deflate",
        "Referer": "http://www.xicidaili.com/wt/1",
    }

    def __init__(self, session_url):
        self.req = requests.session()
        self.req.get(session_url)

    def get_pagesource(self, url):
        '''取得html pagesource'''
        html = self.req.get(url, headers=self.headers)
        return html.content

    def get_all_proxy(self, url, n):
        ''' 取得所有1-n页上的代理IP'''
        data = []
        for i in range(1, n):
            html = self.get_pagesource(url + str(i))
            soup = BeautifulSoup(html, "lxml")

            table = soup.find('table', id="ip_list")
            for row in table.findAll("tr"):
                cells = row.findAll("td")
                tmp = []
                for item in cells:

                    tmp.append(item.find(text=True))
                try:
                    tmp2 = tmp[1:2][0]
                    tmp3 = tmp[2:3][0]
                    tmp4 = tmp[5:6][0]
                    data.append({tmp4: tmp2 + ":" + tmp3})
                except Exception as e:
                    pass
        return data


class IsActivePorxyIP(object):
    """类组合
     用gevent 异步并发验证 代理IP是不是可以用
    """

    def __init__(self, session_url):
        self.proxy = SpiderProxy(session_url)
        self.is_active_proxy_ip = []

    def probe_proxy_ip(self, proxy_ip):
        """代理检测"""
        proxy = urllib.request.ProxyHandler(proxy_ip)
        print(proxy_ip)
        urllib.request.Prox
        opener = urllib.request.build_opener(proxy)
        urllib.request.install_opener(opener)
        try:
            html = urllib.request.urlopen('http://1212.ip138.com/ic.asp')
            #print(html.read().decode("gb2312"))
            if html:
                self.is_active_proxy_ip.append(proxy_ip)
                #print(proxy_ip,"is useful")
                return True
            else:
                return False
        except Exception as e:
            return False


if __name__ == '__main__':
    session_url = 'http://www.xicidaili.com/wt/1'
    url = 'http://www.xicidaili.com/wt/'

    p_isactive = IsActivePorxyIP(session_url)
    proxy_ip_lst = p_isactive.proxy.get_all_proxy(url, 10)

    # 异步并发
    pool = Pool(20)
    pool.map(p_isactive.probe_proxy_ip, proxy_ip_lst)
    #print(p_isactive.is_active_proxy_ip)
```

参考链接：[跟黄哥学Python爬虫抓取代理IP和验证](https://zhuanlan.zhihu.com/p/21648138)

[Python爬虫入门](https://www.zybuluo.com/FadeTrack/note/166205)

[Python:使用代理proxy爬虫](https://platinhom.github.io/2016/01/21/proxy-py/)

但是免费代理失效太快，难以获得合格的IP，只能失败了,只能以后购买收费代理再尝试了。

**代码地址：**[CSDN访问量](https://github.com/whuhan2013/pythoncode/tree/master/csdn)

