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

1、 prompt

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


**2.3 Accessing Parts of a Matrix**       

```
x(2,3) 调出第二行第三个数字
也可以对x(2,3)赋值，则输出改后的整个矩阵
如果对不存在的矩阵中某个位置赋值，则自动生成一个矩阵
x (2,[1,3]) 输出第二行的第一列和第三列 — subarray
获得一个倒过来的矩阵：x(2:-1:1,3:-1:1)
x(end,2)最后一行
x(1,end-1)倒数第二列
x(:,2) 等同于x(1:end,2)
```

**2.4 Combining and Transforming Matrices**     

```
[A1 A2 A3] 横向排列三个矩阵
[A1;A2;A3] 纵向排列三个矩阵
G = H’ 即行列颠倒后的矩阵 (transpose)
(1:2:5)’ 得到1 3 5的三行
```


**2.5 Arithmetic Part 1**  

Z=X+Y是指每个对应位置的元素相加后得到的矩阵    
Z=X.*Y指每个对应位置的元素相乘得到新的矩阵     
Z=X*Y指每个元素乘以行对应列以后的元素的积的和     

Matrix Multiplication (z=x*y) is different from Array Multiplication (z=x.*y):    
     1. Operator has no dot (* instead of .*)                              
     2. Operaands must be “compatible” (as opposed to having the same shape)     
     3. Calculation of each element of z uses both multiplication and addition      
     
[size(A),size(B)] 若得到vector中间两个数相同，则可以做矩阵的乘法

**2.6 Arithmetic Part 2**      

Z=X./Y 每个位置上x/y    
Z=X.\Y 每个位置上y/x           
X.^N 每个位置都开对应位置的N次方     
X^3 X自己乘自己3次——必须是正方形矩阵     
A+3 每个位置上的数都加3       

**Precedence 计算的优先级**

Unary examples: +2, -4, -x, H'    
Binary examples: 3-4, X.*Y, X*Y, A^3      