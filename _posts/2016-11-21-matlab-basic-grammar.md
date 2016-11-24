---
layout: post
title: Matlab基础语法
date: 2016-11-21
categories: blog
tags: [matlab]
description: Matlab基础语法
---

#### 函数     

**3.1 Introduction to Functions**   

- rand(3,4): 生成一个3*4的矩阵，内含0~1的随机数
- 1+rand(3,4)*9: 生成一个3*4的矩阵，内含1~10的随机数
- edit: 打开edit window，可以自己编辑方程

```
Within editor window:
function myRand
a = 1+rand(3,4)*9
end
直接保存后就可以在command window里使用了
```


**3.2 Function I/O**    

就算在editor里用了a变量，而且输出的是a=...但是a并不存在于workspace里，仅仅作输出用——What happens in Vegas, stays in Vegas.
Local variable: a variable that is accessible only by statements inside on function. A local variable exists only during the function call.

```
Within editor window:
function [a, s] = myRand(low,high)
a = low+rand(3,4)*(high-low);
v = a(:); 意思是把a矩阵里的数字全部变成一个行矩阵
s = sum(v);
end
Within command window:
myRand(2,3) 只输出两个输出值里的第一个（即矩阵）
Within command window:
[x ss] = myRand(2,3) 则输出两个输出值
```

**3.3 Formal Definition of Functions**     

function [out_arg1, out_arg2, …] = function_name (in_arg1, in_arg2, …)        
function name should be new & not exist, to check, use help exist       


**3.4 Subfunctions**     

重写上一个公式，用subfunction来实现：

```
Within editor window:
function [a,s] = myRand(low,high)
a = low+ran(3,4)*(high-low);
s = sumAllElements(a);

function summa = sumAllElements(M)
v = M(:);
summa = sum(v);
```

在command window里，只能call主公式，不能call subfunction


**3.5 Scope**     

scope: the set of statements that can access a variable
local scope: accessibility by statements in only one function or only in the Command Window

如何使上一个公式里的v在command window里能够调出：

```
Within editor window:
function [a,s] = myRand(low,high)
a = low+ran(3,4)*(high-low);
s = sumAllElements(a);

function summa = sumAllElements(M)
global v;
v = M(:);
summa = sum(v);
Within command window:
global v 则可输出公式内的v
```

尽量少用global variable，会造成困惑


**3.7 Scripts**     

可以运行的一套程序，公式的一种

```
e.g. 关闭界面
Within command window:
edit EndLessionThree
Within editor window:
fprintf(‘This concludes Lesson 3\n’)
pause(5);
quit;
```

#### Toolbox      

**4.1 Introduction to Programmer’s Toolbox

```
sqrt: 可以开根数字，也可以开根矩阵（将其中每一个元素开根得到新的矩阵）
Polymorphism: having multiple forms
Polymorphism functions example: sqrt (return an output that has the same shape as the input provided)
Other examples of polymorphism:
sum: sum(vector)是每一个元素的和；sum(matrix)是一个行矩阵，每一个元素是原矩阵中每一列的求和(sum is polymorphic, but the shape is not the same as the input)
max: a = max(vector)输出的是一个最大值；[a b] = max(vector)输出的是两个值
size: s = size(matrix)输出的是两个值；[row col] = size(matrix)则分别输出需要的两个值
```

**4.2 Matrix Building**

```
zeros(5,6): 输出一个5*6的0矩阵
ones(4,2): 输出一个4*2的1矩阵
5*ones(4,2): 输出全是5的矩阵
zeros(4): 输出4*4的0矩阵 — polymorphism
diag([7 3 9 1]): 输出对角线是指定数字，其他为0的正方形矩阵
rand: rand输出一个0~1的随机数；rand(3,4)输出3*4的随机数矩阵；rand(5)输出5*5的随机数矩阵
fix(1 + rand(5,4)*10): 输出1~10的随机整数
randi(10,5,4): 输出1~10的随机整数，得到5*4的矩阵
randi(20,5): 输出1~20的随机整数，得到5*5的矩阵 — polymorphism
randi([5 10],2,3): 输出5~10的随机整数，得到2*3的矩阵
randn(5): 输出5*5的矩阵，内含数按照mean=0,deviation=1的正态曲线抽取的
r = randn(1,1000000); hist(r, 100): 输出钟形曲线，横坐标切割为100份。
Matlab生成的随机数其实是Pseudo Random Number，即每次都是一样的数字顺序
rng(0)使得生成的数从第一个（0.8147)开始
rng(1)就是另一套数列——reset rand, randn & randi
rng('shuffle');[rand,randn,randi(10)]: 则可通过shuffle得到真正的随机数
```


**4.3 Input / Output**

```
Edit one_more
Within editor window:
function a = one_more
x = input('Gimme a number, buddy: ');
a = x + 1;
则在操作界面中，自动调入输入的数字，输出加一的值。
fprintf('This concludes Lesson 3\n’): 在command window里print除了\n以外的文本；\n指“Go to a new line”即换行
Edit check_out
Within editor window:
function check_out(n,price)
total = n*price;
fprintf('%d items at %.2f each\nTotal = $%5.2f\n',n,price,total);
Within command window:
check_out(3,2.71)
3 items at 2.71 each
Total = $ 8.13
解释：
%: scape character; 直接指代后面的三个数值——n, price, total
After each % is a format specifier: n&f — a conversion character (how the argument is to print the output)
%d: print using scientific notation, except for whole numbers
%.2f: f means print using fixed point notation instead of scientific notation (normal decimal notation); .2指小数点后保留两位
\n: to start a new line
%5.2f: 5 means to print using at least 5 spaces, in case it only needs 4 spaces, blank spaces will be inserted before digits.
来源于C语言
如何print "%”？
只要在fprintf里输入连续两个%（%%）即可。
例：
fprintf('12.5%% of 1234 equals %.3f\n',0.125*1234)
则输出：12.5% of 1234 equals 154.250
如何print “\”? — “\\"
fprintf('This is a backslash character: \\\n')
则输出：This is a backslash character: \
如何print “ ‘ "? — “ ‘' "
fprintf('How about a single quote ('')?\n')
则输出：How about a single quote (')?
如果有4个%，但是后面的数值只有3个，安全的输出法是输出到第4个%之前，后面的不输出
如果有4个%，但是后面的数值有5个，会重复输出一遍直到第2个（第二次）%之前
若输入fprintf('%4.1f\n',[1 2 3 4 5 6])，则会每一行都print一个数字直到6个结束
```


**4.4 Plotting**

```
figure(2); plot(a)则可对第二个a进行绘图，可以比较两张图（对a赋值一个vector就可以绘图——还会拟合的，但是只有y值，x值是vector的容量比如说1~10）
plot(t,b)就可以自己设定横坐标了
在一个figure上画两个图：
x1 = 0 : 0.1 : 2*pi; y1 = sin(x1);
x2 = pi/2 : 0.1 : 3*pi; y2 = cos(x2);
plot(x1,y1,x2,y2)
如何改变两条曲线的颜色/样式：
plot(x1,y1,'r',x2,y2,'k:’) 则x1-y1是红色实线，x2-y2是黑色虚线
使用doc plot / help plot来查找line styles的代码
另一个方法来在一个figure上画两个图：
>> plot(x1,y1,'r')
>> hold on
>> plot(x2,y2,'k:')
>> hold off
grid: 在figure上显示网格
title, xlabel, ylabel: 添加图表信息
legend(‘aaa’,’bbb’): 添加多条线的指示信息
axis([a b c d]): 四个数字分别是x,y轴的最大/最小值
close all: 关掉所有的figure
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/matlab/p1.png)


