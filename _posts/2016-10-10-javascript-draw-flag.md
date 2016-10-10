---
layout: post
title: Javascript画国旗
date: 2016-10-7
categories: blog
tags: [javascript]
description: Javascript画国旗
---


**适逢国庆佳节，用代码画国旗试试**

```

<html>
<head>
<script>
var col=new Array("red","brown");
var ticker=0;
var step=0;
function drawBackground(){
var g=document.getElementById("background").getContext("2d");
var grd=g.createLinearGradient(-560+ticker, 0, 1400+ticker,0);
for (var i=0; i<10; i++)
grd.addColorStop(i/10,col[(i + step)%col.length]);
ticker=ticker+10;
if (ticker>=196){
    ticker=0;
    step++;
    }
g.fillStyle=grd;
g.fillRect(0,0,1600,700);
}

function preperation(){
    setInterval('drawBackground()',100);
  }
</script>
<style type="text/css">
#myCanvas{
 z-index:2; 
 position:absolute; left:0px; top:-5px;
}
#background{
    z-index:1;
    position:absolute;
    left:0px;
    top:0px;
}
</style>
</head>
<body onLoad="preperation()">
<canvas id="myCanvas" width="900" height="600" >
Your browser does not support the HTML5 canvas tag.
</canvas>
<canvas id="background" width="1600" height="700" ></canvas>
<script>
var x=new Array(0,12,54,18,28,0,-28,-18,-54,-12,0);    //五角星样品坐标xx数组
var y=new Array(-53,-17,-17,1,45,19,45,1,-17,-17,-53); //五角星样品坐标y数组

var c=document.getElementById("myCanvas");
var ctx=c.getContext("2d"); //获得画笔
//样品数组x轴坐标 a , 和y轴坐标 b
//指定位置[locationX,locationY]
//真实五角星的大小，与样品五角星尺寸之比 f
//五角星画完后，旋转的角度 rotation
function star(a,b,locationX,locationY,f,rotation){
ctx.save();//记录画图（画笔）的初始环境
ctx.translate(locationX,locationY);
ctx.rotate(rotation*Math.PI/180.0);
ctx.beginPath();
ctx.moveTo(Math.round(a[0]*f),Math.round(b[0]*f));
for (var i=1;i<a.length;i++)
ctx.lineTo(Math.round(a[i]*f),Math.round(b[i]*f));
ctx.closePath();
ctx.fillStyle="yellow";
ctx.fill();
ctx.restore();//还原画图（画笔）的初始环境
}
star(x,y,165,165,1.4,0);//画大五角星
star(x,y,301,65,0.5,30);//画小五角星
star(x,y,362,126,0.5,-30);//画小五角星
star(x,y,359,216,0.5,0);//画小五角星
star(x,y,301,273,0.5,30);//画小五角星
</script>
</body>
</html>
```

**python版本实现**

```

#coding=utf-8
'''
Created on 2014-11-17
 
@author: Neo
'''
import turtle
import math
 
def draw_polygon(aTurtle, size=50, n=3):
    ''' 绘制正多边形
 
    args:
        aTurtle: turtle对象实例
        size: int类型，正多边形的边长
        n: int类型，是几边形        
    '''
    for i in xrange(n):
        aTurtle.forward(size)
        aTurtle.left(360.0/n)
 
def draw_n_angle(aTurtle, size=50, num=5, color=None):
    ''' 绘制正n角形，默认为黄色
 
    args:
        aTurtle: turtle对象实例
        size: int类型，正多角形的边长
        n: int类型，是几角形    
        color: str， 图形颜色，默认不填色
    '''
    if color:
        aTurtle.begin_fill()
        aTurtle.fillcolor(color)
    for i in xrange(num):
        aTurtle.forward(size)
        aTurtle.left(360.0/num)
        aTurtle.forward(size)
        aTurtle.right(2*360.0/num)
    if color:
        aTurtle.end_fill()
 
def draw_5_angle(aTurtle=None, start_pos=(0,0), end_pos=(0,10), radius=100, color=None):
    ''' 根据起始位置、结束位置和外接圆半径画五角星
 
    args:
        aTurtle: turtle对象实例
        start_pos: int的二元tuple，要画的五角星的外接圆圆心
        end_pos: int的二元tuple，圆心指向的位置坐标点
        radius: 五角星外接圆半径
        color: str， 图形颜色，默认不填色    
    '''
    aTurtle = aTurtle or turtle.Turtle()
    size = radius * math.sin(math.pi/5)/math.sin(math.pi*2/5)
    aTurtle.left(math.degrees(math.atan2(end_pos[1]-start_pos[1], end_pos[0]-start_pos[0])))
    aTurtle.penup()
    aTurtle.goto(start_pos)
    aTurtle.fd(radius)
    aTurtle.pendown()
    aTurtle.right(math.degrees(math.pi*9/10))
    draw_n_angle(aTurtle, size, 5, color)
 
def draw_5_star_flag(times=20.0):
    ''' 绘制五星红旗
 
    args:
        times: 五星红旗的规格为30*20， times为倍数，默认大小为10倍， 即300*200
    '''
    width, height = 30*times, 20*times
    # 初始化屏幕和海龟
    window = turtle.Screen()
    aTurtle = turtle.Turtle()
    aTurtle.hideturtle()
    aTurtle.speed(10)
    # 画红旗
    aTurtle.penup()
    aTurtle.goto(-width/2, height/2)
    aTurtle.pendown()
    aTurtle.begin_fill()
    aTurtle.fillcolor('red')
    aTurtle.fd(width)
    aTurtle.right(90)
    aTurtle.fd(height)
    aTurtle.right(90)
    aTurtle.fd(width)
    aTurtle.right(90)
    aTurtle.fd(height)
    aTurtle.right(90)    
    aTurtle.end_fill()
    # 画大星星
    draw_5_angle(aTurtle, start_pos=(-10*times, 5*times), end_pos=(-10*times, 8*times), radius=3*times, color='yellow')  
    # 画四个小星星
    stars_start_pos = [(-5, 8), (-3, 6), (-3, 3), (-5, 1)]
    for pos in stars_start_pos:
        draw_5_angle(aTurtle, start_pos=(pos[0]*times, pos[1]*times), end_pos=(-10*times, 5*times), radius=1*times, color='yellow')  
    # 点击关闭窗口
    window.exitonclick()
 
if __name__ == '__main__':
    draw_5_star_flag()
```

除了javascript,python版本，还有java，c++,ruby等版本      
详见:[国庆节，我们用代码来画个国旗](https://zhuanlan.zhihu.com/p/22754747)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p1.png)








