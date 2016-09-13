---
layout: post
title: python异步爬虫 
date: 2016-9-13
categories: blog
tags: [网络爬虫]
description: python异步爬虫 
---

**本文主要包括以下内容**　　　　　　　　　　　

- 线程池实现并发爬虫
- 回调方法实现异步爬虫
- 协程技术的介绍
- 一个基于协程的异步编程模型
- 协程实现异步爬虫


**线程池、回调、协程**

我们希望通过并发执行来加快爬虫抓取页面的速度。一般的实现方式有三种：

- 线程池方式：开一个线程池，每当爬虫发现一个新链接，就将链接放入任务队列中，线程池中的线程从任务队列获取一个链接，之后建立socket，完成抓取页面、解析、将新连接放入工作队列的步骤。
- 回调方式：程序会有一个主循环叫做事件循环，在事件循环中会不断获得事件，通过在事件上注册解除回调函数来达到多任务并发执行的效果。缺点是一旦需要的回调操作变多，代码就会非常散，变得难以维护。
- 协程方式：同样通过事件循环执行程序，利用了Python 的生成器特性，生成器函数能够中途停止并在之后恢复，那么原本不得不分开写的回调函数就能够写在一个生成器函数中了，这也就实现了协程。



#### 线程池实现爬虫

**python多线程建立线程的两种方式**


```
#第一种:通过函数创建线程
def 函数a():
    pass
t = threading.Thread(target=函数a,name=自己随便取的线程名字)

#第二种:继承线程类
class Fetcher(threading.Thread):
    def __init__(self):
        Thread.__init__(self):
        #加这一步后主程序中断退出后子线程也会跟着中断退出
        self.daemon = True
    def run(self):
        #线程运行的函数
        pass
t = Fetcher()
```

**参见：**[python 多线程就这么简单 - 虫师 - 博客园](http://www.cnblogs.com/fnng/p/3670789.html)


**多线程同步－队列**

多线程同步就是多个线程竞争一个全局变量时按顺序读写，一般情况下要用锁，但是使用标准库里的Queue的时候它内部已经实现了锁，不用程序员自己写了。

导入队列类：

```
from queue import Queue
创建一个队列：

q = Queue(maxsize=0)
maxsize为队列大小，为0默认队列大小可无穷大。

队列是先进先出的数据结构：

q.put(item) #往队列添加一个item，队列满了则阻塞
q.get(item) #从队列得到一个item，队列为空则阻塞
还有相应的不等待的版本，这里略过。

队列不为空，或者为空但是取得item的线程没有告知任务完成时都是处于阻塞状态

q.join()    #阻塞直到所有任务完成
线程告知任务完成使用task_done

q.task_done() #在线程内调用
```

**完整代码**

```
from queue import Queue 
from threading import Thread, Lock
import urllib.parse
import socket
import re
import time

seen_urls = set(['/'])
lock = Lock()


class Fetcher(Thread):
    def __init__(self, tasks):
        Thread.__init__(self)
        self.tasks = tasks
        self.daemon = True

        self.start()

    def run(self):
        while True:
            url = self.tasks.get()
            print(url)
            sock = socket.socket()
            sock.connect(('localhost', 3000))
            get = 'GET {} HTTP/1.0\r\nHost: localhost\r\n\r\n'.format(url)
            sock.send(get.encode('ascii'))
            response = b''
            chunk = sock.recv(4096)
            while chunk:
                response += chunk
                chunk = sock.recv(4096)

            links = self.parse_links(url, response)

            lock.acquire()
            for link in links.difference(seen_urls):
                self.tasks.put(link)
            seen_urls.update(links)    
            lock.release()

            self.tasks.task_done()

    def parse_links(self, fetched_url, response):
        if not response:
            print('error: {}'.format(fetched_url))
            return set()
        if not self._is_html(response):
            return set()
        urls = set(re.findall(r'''(?i)href=["']?([^\s"'<>]+)''',
                              self.body(response)))

        links = set()
        for url in urls:
            normalized = urllib.parse.urljoin(fetched_url, url)
            parts = urllib.parse.urlparse(normalized)
            if parts.scheme not in ('', 'http', 'https'):
                continue
            host, port = urllib.parse.splitport(parts.netloc)
            if host and host.lower() not in ('localhost'):
                continue
            defragmented, frag = urllib.parse.urldefrag(parts.path)
            links.add(defragmented)

        return links

    def body(self, response):
        body = response.split(b'\r\n\r\n', 1)[1]
        return body.decode('utf-8')

    def _is_html(self, response):
        head, body = response.split(b'\r\n\r\n', 1)
        headers = dict(h.split(': ') for h in head.decode().split('\r\n')[1:])
        return headers.get('Content-Type', '').startswith('text/html')


class ThreadPool:
    def __init__(self, num_threads):
        self.tasks = Queue()
        for _ in range(num_threads):
            Fetcher(self.tasks)

    def add_task(self, url):
        self.tasks.put(url)

    def wait_completion(self):
        self.tasks.join()

if __name__ == '__main__':
    start = time.time()
    pool = ThreadPool(4)
    pool.add_task("/")
    pool.wait_completion()
    print('{} URLs fetched in {:.1f} seconds'.format(len(seen_urls),time.time() - start))
```


#### 事件驱动-回调函数实现爬虫

**非阻塞I/O**

如果使用非阻塞I/O，程序就不会傻傻地等在那里(比如等连接、等读取)，而是会返回一个错误信息，虽然说是说错误信息，它其实就是叫你过一会再来的意思，编程的时候都不把它当错误看。

非阻塞I/O代码如下：

```
sock = socket.socket()
sock.setblocking(False)
try:
    sock.connect(('xkcd.com', 80))
except BlockingIOError:
    pass
```

**单线程上的多I/O**

有了非阻塞I/O这个特性，我们就能够实现单线程上多个sockets的处理了，学过C语言网络编程的同学应该都认识select这个函数吧？不认识也不要紧，select函数如果你不设置它的超时时间它就是默认一直阻塞的，只有当有I/O事件发生时它才会被激活，然后告诉你哪个socket上发生了什么事件（读|写|异常），在Python中也有select，还有跟select功能相同但是更高效的poll，它们都是底层C函数的Python实现。

不过这里我们不使用select，而是用更简单好用的DefaultSelector，是Python 3.4后才出现的一个模块里的类，你只需要在非阻塞socket和事件上绑定回调函数就可以了。

代码如下：

```
from selectors import DefaultSelector, EVENT_WRITE

selector = DefaultSelector()

sock = socket.socket()
sock.setblocking(False)
try:
    sock.connect(('localhost', 3000))
except BlockingIOError:
    pass

def connected():
    selector.unregister(sock.fileno())
    print('connected!')

selector.register(sock.fileno(), EVENT_WRITE, connected)
```

这里看一下selector.register的原型

register(fileobj, events, data=None)
其中fileobj可以是文件描述符也可以是文件对象（通过fileno得到），events是位掩码，指明发生的是什么事件，data 则是与指定文件（也就是我们的socket）与指定事件绑定在一起的数据。

如代码所示，selector.register 在该socket的写事件上绑定了回调函数connected（这里作为数据绑定）。在该socket上第一次发生的写事件意味着连接的建立，connected函数在连接建立成功后再解除了该socket上所有绑定的数据。


**事件驱动**

看了以上selector的使用方式，我想你会发现它很适合写成事件驱动的形式。

我们可以创建一个事件循环，在循环中不断获得I/O事件：

```
def loop():
    while True:
     events = selector.select()
        #遍历事件并调用相应的处理
        for event_key, event_mask in events:
         callback = event_key.data
            callback()
```


完整代码


```
from selectors import *
import socket
import re
import urllib.parse
import time


urls_todo = set(['/'])
seen_urls = set(['/'])
#追加了一个可以看最高并发数的变量
concurrency_achieved = 0
selector = DefaultSelector()
stopped = False


class Fetcher:
    def __init__(self, url):
        self.response = b''
        self.url = url
        self.sock = None

    def fetch(self):
        global concurrency_achieved
        concurrency_achieved = max(concurrency_achieved, len(urls_todo))

        self.sock = socket.socket()
        self.sock.setblocking(False)
        try:
            self.sock.connect(('localhost', 3000))
        except BlockingIOError:
            pass
        selector.register(self.sock.fileno(), EVENT_WRITE, self.connected)

    def connected(self, key, mask):
        selector.unregister(key.fd)
        get = 'GET {} HTTP/1.0\r\nHost: localhost\r\n\r\n'.format(self.url)
        self.sock.send(get.encode('ascii'))
        selector.register(key.fd, EVENT_READ, self.read_response)

    def read_response(self, key, mask):
        global stopped

        chunk = self.sock.recv(4096)  # 4k chunk size.
        if chunk:
            self.response += chunk
        else:
            selector.unregister(key.fd)  # Done reading.
            links = self.parse_links()
            for link in links.difference(seen_urls):
                urls_todo.add(link)
                Fetcher(link).fetch()

            seen_urls.update(links)
            urls_todo.remove(self.url)
            if not urls_todo:
                stopped = True
            print(self.url)

    def body(self):
        body = self.response.split(b'\r\n\r\n', 1)[1]
        return body.decode('utf-8')

    def parse_links(self):
        if not self.response:
            print('error: {}'.format(self.url))
            return set()
        if not self._is_html():
            return set()
        urls = set(re.findall(r'''(?i)href=["']?([^\s"'<>]+)''',
                              self.body()))

        links = set()
        for url in urls:
            normalized = urllib.parse.urljoin(self.url, url)
            parts = urllib.parse.urlparse(normalized)
            if parts.scheme not in ('', 'http', 'https'):
                continue
            host, port = urllib.parse.splitport(parts.netloc)
            if host and host.lower() not in ('localhost'):
                continue
            defragmented, frag = urllib.parse.urldefrag(parts.path)
            links.add(defragmented)

        return links

    def _is_html(self):
        head, body = self.response.split(b'\r\n\r\n', 1)
        headers = dict(h.split(': ') for h in head.decode().split('\r\n')[1:])
        return headers.get('Content-Type', '').startswith('text/html')


start = time.time()
fetcher = Fetcher('/')
fetcher.fetch()

while not stopped:
    events = selector.select()
    for event_key, event_mask in events:
        callback = event_key.data
        callback(event_key, event_mask)

print('{} URLs fetched in {:.1f} seconds, achieved concurrency = {}'.format(
    len(seen_urls), time.time() - start, concurrency_achieved))
```


### 事件驱动-协程实现爬虫

**什么是协程？**

协程其实是比起一般的子例程而言更宽泛的存在，子例程是协程的一种特例。

子例程的起始处是惟一的入口点，一旦退出即完成了子例程的执行，子例程的一个实例只会返回一次。

协程可以通过yield来调用其它协程。通过yield方式转移执行权的协程之间不是调用者与被调用者的关系，而是彼此对称、平等的。

协程的起始处是第一个入口点，在协程里，返回点之后是接下来的入口点。子例程的生命期遵循后进先出（最后一个被调用的子例程最先返回）；相反，协程的生命期完全由他们的使用的需要决定。

还记得我们什么时候会用到yield吗，就是在生成器(generator)里，在迭代的时候每次执行next(generator)生成器都会执行到下一次yield的位置并返回，可以说生成器就是例程。


#### 生成器实现协程模型

虽然生成器拥有一个协程该有的特性，但光这样是不够的，做异步编程仍是困难的，我们需要先用生成器实现一个协程异步编程的简单模型，它同时也是Python标准库asyncio的简化版，正如asyncio的实现，我们会用到生成器,Future类，以及yield from语句。

首先实现Future类， Future类可以认为是专门用来存储将要发送给协程的信息的类。

```
class Future:
    def __init__(self):
        self.result = None
        self._callbacks = []

    def add_done_callback(self, fn):
        self._callbacks.append(fn)

    def set_result(self, result):
        self.result = result
        for fn in self._callbacks:
            fn(self)
```


Future对象最开始处在挂起状态，当调用set_result时被激活，并运行注册的回调函数，该回调函数多半是对协程发送信息让协程继续运行下去的函数。

我们改造一下之前从fetch到connected的函数，加入Future与yield。

这是之前回调实现的fetch：

```
class Fetcher:
    def fetch(self):
        self.sock = socket.socket()
        self.sock.setblocking(False)
        try:
            self.sock.connect(('localhost', 3000))
        except BlockingIOError:
            pass
        selector.register(self.sock.fileno(),
                          EVENT_WRITE,
                          self.connected)

    def connected(self, key, mask):
        print('connected!')
        # ...后面省略...
```

改造后，我们将连接建立后的部分也放到了fetch中。

```
class Fetcher:
    def fetch(self):
        sock = socket.socket()
        sock.setblocking(False)
        try:
            sock.connect(('localhost', 3000))
        except BlockingIOError:
            pass

        f = Future()

        def on_connected():
            #连接建立后通过set_result协程继续从yield的地方往下运行
            f.set_result(None)

        selector.register(sock.fileno(),
                          EVENT_WRITE,
                          on_connected)
        yield f
        selector.unregister(sock.fileno())
        print('connected!')
```

fetcher是一个生成器函数，我们创建一个Future实例，yield它来暂停fetch的运行直到连接建立f.set_result(None)的时候，生成器才继续运行。那set_result时运行的回调函数是哪来的呢？这里引入Task类：

```
class Task:
    def __init__(self, coro):
        #协程
        self.coro = coro
        #创建并初始化一个为None的Future对象
        f = Future()
        f.set_result(None)
        #步进一次（发送一次信息）
        #在初始化的时候发送是为了协程到达第一个yield的位置，也是为了注册下一次的步进
        self.step(f)

    def step(self, future):
        try:
            #向协程发送消息并得到下一个从协程那yield到的Future对象
            next_future = self.coro.send(future.result)
        except StopIteration:
            return

        next_future.add_done_callback(self.step)

fetcher = Fetcher('/')
Task(fetcher.fetch())

loop()
```

流程大致是这样的，首先Task初始化，向fetch生成器发送None信息（也可以想象成step调用了fetch，参数是None），fetch得以从开头运行到第一个yield的地方并返回了一个Future对象给step的next_future，然后step就在这个得到的Future对象注册了step。当连接建立时on_connected就会被调用，再一次向协程发送信息，协程就会继续往下执行了。


**使用yield from分解协程**

一旦socket连接建立成功，我们发送HTTP GET请求到服务器并在之后读取服务器响应。现在这些步骤不用再分散在不同的回调函数里了，我们可以将其放在同一个生成器函数中：

```
def fetch(self):
    # ... 省略连接的代码
    sock.send(request.encode('ascii'))

    while True:
        f = Future()

        def on_readable():
            f.set_result(sock.recv(4096))

        selector.register(sock.fileno(),
                          EVENT_READ,
                          on_readable)
        chunk = yield f
        selector.unregister(sock.fileno())
        if chunk:
            self.response += chunk
        else:
            # 完成读取
            break
```

但是这样代码也会越积越多，可不可以分解生成器函数的代码呢，从协程中提取出子协程？Python 3 的yield from能帮助我们完成这部分工作。：

```
>>> def gen_fn():
...     result = yield 1
...     print('result of yield: {}'.format(result))
...     result2 = yield 2
...     print('result of 2nd yield: {}'.format(result2))
...     return 'done'
...
```

yield from得到的子协程最后return的返回值

**完整代码**

```
from selectors import *
import socket
import re
import urllib.parse
import time


class Future:
    def __init__(self):
        self.result = None
        self._callbacks = []

    def result(self):
        return self.result

    def add_done_callback(self, fn):
        self._callbacks.append(fn)

    def set_result(self, result):
        self.result = result
        for fn in self._callbacks:
            fn(self)

    def __iter__(self):
        yield self 
        return self.result


class Task:
    def __init__(self, coro):
        self.coro = coro
        f = Future()
        f.set_result(None)
        self.step(f)

    def step(self, future):
        try:
            next_future = self.coro.send(future.result)
        except StopIteration:
            return

        next_future.add_done_callback(self.step)


urls_seen = set(['/'])
urls_todo = set(['/'])
#追加了一个可以看最高并发数的变量
concurrency_achieved = 0
selector = DefaultSelector()
stopped = False


def connect(sock, address):
    f = Future()
    sock.setblocking(False)
    try:
        sock.connect(address)
    except BlockingIOError:
        pass

    def on_connected():
        f.set_result(None)

    selector.register(sock.fileno(), EVENT_WRITE, on_connected)
    yield from f
    selector.unregister(sock.fileno())


def read(sock):
    f = Future()

    def on_readable():
        f.set_result(sock.recv(4096))  # Read 4k at a time.

    selector.register(sock.fileno(), EVENT_READ, on_readable)
    chunk = yield from f
    selector.unregister(sock.fileno())
    return chunk


def read_all(sock):
    response = []
    chunk = yield from read(sock)
    while chunk:
        response.append(chunk)
        chunk = yield from read(sock)

    return b''.join(response)


class Fetcher:
    def __init__(self, url):
        self.response = b''
        self.url = url

    def fetch(self):
        global concurrency_achieved, stopped
        concurrency_achieved = max(concurrency_achieved, len(urls_todo))

        sock = socket.socket()
        yield from connect(sock, ('localhost', 3000))
        get = 'GET {} HTTP/1.0\r\nHost: localhost\r\n\r\n'.format(self.url)
        sock.send(get.encode('ascii'))
        self.response = yield from read_all(sock)

        self._process_response()
        urls_todo.remove(self.url)
        if not urls_todo:
            stopped = True
        print(self.url)

    def body(self):
        body = self.response.split(b'\r\n\r\n', 1)[1]
        return body.decode('utf-8')



    def _process_response(self):
        if not self.response:
            print('error: {}'.format(self.url))
            return
        if not self._is_html():
            return
        urls = set(re.findall(r'''(?i)href=["']?([^\s"'<>]+)''',
                              self.body()))

        for url in urls:
            normalized = urllib.parse.urljoin(self.url, url)
            parts = urllib.parse.urlparse(normalized)
            if parts.scheme not in ('', 'http', 'https'):
                continue
            host, port = urllib.parse.splitport(parts.netloc)
            if host and host.lower() not in ('localhost'):
                continue
            defragmented, frag = urllib.parse.urldefrag(parts.path)
            if defragmented not in urls_seen:
                urls_todo.add(defragmented)
                urls_seen.add(defragmented)
                Task(Fetcher(defragmented).fetch())

    def _is_html(self):
        head, body = self.response.split(b'\r\n\r\n', 1)
        headers = dict(h.split(': ') for h in head.decode().split('\r\n')[1:])
        return headers.get('Content-Type', '').startswith('text/html')


start = time.time()
fetcher = Fetcher('/')
Task(fetcher.fetch())

while not stopped:
    events = selector.select()
    for event_key, event_mask in events:
        callback = event_key.data
        callback()

print('{} URLs fetched in {:.1f} seconds, achieved concurrency = {}'.format(
    len(urls_seen), time.time() - start, concurrency_achieved))
```


#### 总结

至此，我们在学习的过程中掌握了：

- 线程池实现并发爬虫
- 回调方法实现异步爬虫
- 协程技术的介绍
- 一个基于协程的异步编程模型
- 协程实现异步爬虫

三种爬虫的实现方式中线程池是最坏的选择，因为它既占用内存，又有线程竞争的危险需要程序员自己编程解决，而且产生的I/O阻塞也浪费了CPU占用时间。再来看看回调方式，它是一种异步方法，所以I/O阻塞的问题解决了，而且它是单线程的不会产生竞争，问题好像都解决了。然而它引入了新的问题，它的问题在于以这种方式编写的代码不好维护，也不容易debug。看来协程才是最好的选择，我们实现的协程异步编程模型使得一个单线程能够很容易地改写为协程。那是不是每一次做异步编程都要实现Task、Future呢？不是的，你可以直接使用asyncio官方标准协程库，它已经帮你把Task、Future封装好了，你根本不会感受到它们的存在，是不是很棒呢？如果你使用Python 3.5那更好，已经可以用原生的协程了，Python 3.5追加了async def，await等协程相关的关键词。


**参考链接：**[Python - Python实现基于协程的异步爬虫 - 实验楼](https://www.shiyanlou.com/courses/574)

