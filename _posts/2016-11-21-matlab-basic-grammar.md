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

