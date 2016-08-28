---
layout: post
title: python小游戏实现
date: 2016-8-25
categories: blog
tags: [python]
description: 
---


**本文主要实现了以下小游戏**        

1. 五子棋      
2. 贪吃蛇AI


#### 五子棋  
五子棋用python代码实现只需要87行，非常简洁，用到了[graphics.py](http://download.csdn.net/detail/w1135181854u/6730647)库,下载后放到python中的lib文件夹中即可  

```
from graphics import*
p=[[0 for a in range(16)] for b in range(16)]
black=[[0 for a in range(16)] for b in range(16)]
white=[[0 for a in range(16)] for b in range(16)]
q=[[0 for a in range(15)] for b in range(15)]

win=GraphWin('wuziqi',480,600)
def WinBoard():
    for i in range(15):
       for j in range(15):
            p[i][j]=Point(i*30+30,j*30+30)
            p[i][j].draw(win)
    for r in range(15):
        Line(p[r][0],p[r][14]).draw(win)
    for s in range(15):
        Line(p[0][s],p[14][s]).draw(win)
    center=Circle(p[7][7],3)
    center.draw(win)
    center.setFill('black')

def Click():
    p1=win.getMouse()
    x1=p1.getX()
    y1=p1.getY()
    for i in range(15):
        for j in range(15):
            sqrdis=((x1-p[i][j].getX())*(x1-p[i][j].getX()))+(y1-p[i][j].getY())*(y1-p[i][j].getY())
            if sqrdis<=200 and q[i][j]==0:
               if p[15][15]%2==0:
                    black[i+1][j+1]=1
                    q[i][j]=Circle(p[i][j],10)
                    q[i][j].draw(win)      
                    q[i][j].setFill('black')
               if p[15][15]%2==1:
                    white[i+1][j+1]=1
                    q[i][j]=Circle(p[i][j],10)
                    q[i][j].draw(win)
                    q[i][j].setFill('white')
               p[15][15]+=1

def Check():
    for i in range(15):
        for j in range(11):
            if black[i+1][j+1] and black[i+1][j+2] and black[i+1][j+3] and black[i+1][j+4] and black[i+1][j+5]:
               return 1
               break
            if white[i+1][j+1] and white[i+1][j+2]and white[i+1][j+3]and white[i+1][j+4]and white[i+1][j+5]:
               return 2
               break
    for i in range(11):
        for j in range(15):
            if black[i+1][j+1] and black[i+2][j+1]and black[i+3][j+1]and black[i+4][j+1]and black[i+5][j+1]:
               return 1
               break
            if white[i+1][j+1] and white[i+2][j+1]and white[i+3][j+1]and white[i+4][j+1]and white[i+5][j+1]:
               return 2
               break
    for i in range(11):
        for j in range(11):
            if black[i+1][j+1] and black[i+2][j+2]and black[i+3][j+3]and black[i+4][j+4]and black[i+5][j+5]:
               return 1
               break
            if white[i+1][j+1] and white[i+2][j+2]and white[i+3][j+3]and white[i+4][j+4]and white[i+5][j+5]:
               return 2
               break
    for i in range(11):
        for j in range(15):
            if black[i+1][j+1] and black[i+2][j]and black[i+3][j-1]and black[i+4][j-2]and black[i+5][j-3]:
               return 1
               break
            if white[i+1][j+1] and white[i+2][j]and white[i+3][j-1]and white[i+4][j-2]and white[i+5][j-3]:
               return 2
               break

def main():
    WinBoard()
    while 1:
        Click()
        Check()
        if Check()==1:
            Text(Point(240,550),'the black wins').draw(win)
            break
        if Check()==2:
            Text(Point(240,550),'the white wins').draw(win)
            break
    win.getMouse()
main()
```

**源码地址**           
[python大作业 五子棋 人人对战 - 下载频道 - CSDN.NET](http://download.csdn.net/detail/w1135181854u/6730681)

**效果如下**   

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/p1.png)



#### 贪吃蛇AI

用到了广度优先遍历等知识及curses模块

详见：[如何用Python写一个贪吃蛇AI](http://www.hawstein.com/posts/snake-ai.html)

原版用的是python2编写，在python3中运行需要一些改动:[Python 2.7.x 和 Python 3.x 的主要区别](https://segmentfault.com/a/1190000000618286#xrange)

**效果如下**     

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/p2.png)
