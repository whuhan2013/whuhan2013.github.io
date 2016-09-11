---
layout: post
title: python简单聊天室实现
date: 2016-9-11
categories: blog
tags: [python]
description: python简单聊天室
---

用python的socket模块实现了一个聊天室的程序
虽然功能比较简单，但是该有的基本功能还是有的

**服务端**   

```
import socket
import threading

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

sock.bind(('localhost', 5550))

sock.listen(5)
print('Server', socket.gethostbyname('localhost'), 'listening ...')

mydict = dict()
mylist = list()


# 把whatToSay传给除了exceptNum的所有人
def tellOthers(exceptNum, whatToSay):
    for c in mylist:
        if c.fileno() != exceptNum:
            try:
                c.send(whatToSay.encode())
            except:
                pass


def subThreadIn(myconnection, connNumber):
    nickname = myconnection.recv(1024).decode()
    mydict[myconnection.fileno()] = nickname
    mylist.append(myconnection)
    print('connection', connNumber, ' has nickname :', nickname)
    tellOthers(connNumber, '【系统提示：' + mydict[connNumber] + ' 进入聊天室】')
    while True:
        try:
            recvedMsg = myconnection.recv(1024).decode()
            if recvedMsg:
                print(mydict[connNumber], ':', recvedMsg)
                tellOthers(connNumber, mydict[connNumber] + ' :' + recvedMsg)

        except (OSError, ConnectionResetError):
            print('here')
            try:
                mylist.remove(myconnection)
            except:
                pass


            print(mydict[connNumber], 'exit, ', len(mylist), ' person left')
            tellOthers(connNumber, '【系统提示：' + mydict[connNumber] + ' 离开聊天室】')
            break
            myconnection.close()
            return


while True:
    connection, addr = sock.accept()
    print('Accept a new connection', connection.getsockname(), connection.fileno())
    try:
        # connection.settimeout(5)
        buf = connection.recv(1024).decode()
        if buf == '1':
            connection.send(b'welcome to server!')

            # 为当前连接开辟一个新的线程
            mythread = threading.Thread(target=subThreadIn, args=(connection, connection.fileno()))
            mythread.setDaemon(True)
            mythread.start()

        else:
            connection.send(b'please go out!')
            connection.close()
    except:
        pass

```


**客户端程序**            


```
import socket
import time
import threading

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

#sock.connect(('localhost',5550))
sock.connect(('119.29.152.242', 5550))
sock.send(b'1')
print(sock.recv(1024).decode())
nickName = input('input your nickname: ')
sock.send(nickName.encode())


def sendThreadFunc():
    while True:
        try:
            myword = input()
            sock.send(myword.encode())
            # print(sock.recv(1024).decode())
        except ConnectionAbortedError:
            print('Server closed this connection!')
        except ConnectionResetError:
            print('Server is closed!')


def recvThreadFunc():
    while True:
        try:
            otherword = sock.recv(1024)
            if otherword:
                print(otherword.decode())
            else:
                pass
        except ConnectionAbortedError:
            print('Server closed this connection!')

        except ConnectionResetError:
            print('Server is closed!')


th1 = threading.Thread(target=sendThreadFunc)
th2 = threading.Thread(target=recvThreadFunc)
threads = [th1, th2]

for t in threads:
    t.setDaemon(True)
    t.start()
t.join()
```

**参考链接：**[Python socket聊天室程序](http://blog.csdn.net/Calling_Wisdom/article/details/42524745)

**源代码：**[聊天室](https://github.com/whuhan2013/pythoncode/tree/master/TalkRoom)

**效果如下**  

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/p4.png)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/p5.png)