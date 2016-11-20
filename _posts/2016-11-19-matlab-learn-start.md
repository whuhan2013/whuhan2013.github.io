---
layout: post
title: Matlab学习入门
date: 2016-11-19
categories: blog
tags: [matlab]
description: Matlab学习入门
---

**下载与安装**     
[Mac系统安装MATLAB 2015b 破解版](http://blog.csdn.net/huangfei711/article/details/52194944)    

也可以使用matlabOnLine,在coursera上注册范德堡大学的 Introduction to Programming with MATLAB可以获取在线版
matlab的lincense       
[https://matlab.mathworks.com/](https://matlab.mathworks.com/)

1. prompt

a symbol, or symbols, printed by a program to indicate that it's ready 4 input from the user of the program.

2、 Ctrl + C

powerful command to quit



3、CLC

clean the screen



4、variable

once assigned value, store it until change



5、function = command



6、up button

show the latest command history or repeat command



7、变量名可以使用underscore、upper case（任何位置）、数字，但数字不能放在第一个、特殊符号也不能出现在变量名，下划线不能出现在第一个位置



8、一行可以有两个分号隔开的commands，多行可以构成一个command，在前一行加上 ...



9、syntax is a language's set of rules for the form of the statements.



10、 semantics is the meaning.

比如x和y交换值的常见错误 x = y; y=x; %这样没有起到交换的本意

```
>> temp = x;        
>> x = y;      
>> y = temp;       
```


11、 语义错误比语法错误更严重，且不易发现



12.format 显示输出形式 具体用法可以参照help



13.help是非常有用的，可以用help 不知具体用法的函数/ doc 不知具体用法的函数。区别在于doc是新建了一个窗口显示。



14.plot函数的参数 （argument） 可以通过help来查找具体用法。



15.axis on/off 是坐标轴是否显示，grid on/off 是网格是否显示。

#### 矩阵与运算符      

Array: 列阵

2D Array: Matrix

1D Array: Vector



Arrays —> matrices —> vectors —> scalars (1*1 matrix / numbers)

xvalues = [1,2,10] — Vector

X = [0 1 -1; 2.5 pi 100] — Matrix


**The Colon Operator**

```
x = 1: 3: 7 (means starting from 1, increasing 3 at a time, go no higher than 7)

x =

1 4 7

ints = 1:100
得到一百个连续整数列

和colon(1,3,7) / colon(1,100)是一样的
```

