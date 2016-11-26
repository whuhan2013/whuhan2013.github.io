---
layout: post
title: Matlab基础语法(二)
date: 2016-11-24
categories: blog
tags: [matlab]
description: Matlab基础语法
---


**本文主要包括以下内容**    

- 选择   
- 循环  
- 文件输入输出   
- 数据类型     

#### 选择    

**5.1 Selection**

```
If-Statement:
Edit guess_my_number
function guess_my_number(x)
if x == 2
fprintf('Congrats! You guessed my number!\n');
end
end
x == 2中的==表示等于，一般的=表示赋值
If-else-statement
function guess_my_number(x)
if x == 2
fprintf('Congrats! You guessed my number!\n');
else
fprintf('Not right, but a good guess.\n');
end
end
If-elseif-else-statement
function guess_my_number(x)
if x == 42
fprintf('Congrats! You guessed my number!\n');
elseif x < 42
fprintf('Too small. Try again.\n');
else
fprintf('Too big. Try again.\n');
end
end
You can have how many “else-ifs" as you want.
```


**5.2 If-Statements, continued**


Return: Return control to invoking function        
This MATLAB function forces MATLAB to return control to the invoking function before it reaches the end of the function.
在function里面可以将得到的结果重新运用于上一个if语句


**5.3 Relational and Logical Operators**

```
Relational Operators:
==: is equal to
~=: is not equal to
>: is greater than
<: is less than
>=: is greater than or equal to
<=: is less than or equal to

Logical operators:
&&: and (true only if both operants are true)
||: or (false only if both operants are false)
~: not (true->false; false->true)
```

**5.4 Nested If-Statements**

```
If-statements can contain other if-statements
More flexible to categorize & combine things in multiple ways.
```

**5.5 Variable Number of Function Arguments**

```
How do we make our functions polymorphic? — Use if-statments!
Two built-in functions:
nargin: returns the number of actual input arguments that the function was called with
nargout: returns the number of output arguments that the function caller requested
```


**5.6 Robustness**

```
A function declaration specifies:
Name of the function,
Number of input arguments, and
Number of output arguments
Function code and documentation specify:
What the function does, and
The type of the arguments
What the arguments represent
Robustness
A function is robust if it handles erroneous input and output arguments, and
Provides a meaningful error message
Comments
Extra text that is not part of the code
MATLAB disregards it
Anything after a % is a comment until the end of the line
Purpose:
Extra information for human reader
Explain important or complicated parts of the program
Provide documentation of your functions
Comments right after the function declaration are used by the built-in help function

isscalar(m): 判断m是否为scalar
error(‘…’): 输出错误提示并停止运算
```

**5.7 Persistent Variables**

```
Variables:
Local
Global
Persistent
Persistent variable:
It’s a local variable, but its value persists from one call of the function to the next.
Relatively rarely used
None of the bad side effects of global variables.


Example:
edit accumulate
function total = accumulate(n)
persistent summa;
if isempty(summa)
summa = n;
else
summa = summa + n;
end
total = summa;
可以不断地把n累加上去；每一次运算都会记住上一次的值
如要清除persistent variable，只需输入clear accumulate (function name)即可
Example: 重新修正multable（逗比报错版）
edit snarky_multable
function [table summa] = snarky_multable(n,m)
```


#### 循环      

**6.1 For-Loops**

```
The loop is a new control construct that makes it possible to repeat a block of statements a number of times.
We have already used loops without knowing it:

matlab里矩阵的点运算（.* etc）都是implicit loop
斐波那契数列
edit fibo
function f = fibo(n)
if ( ~isscalar(n) || n < 1 || n ~= fix(n))
error('n must be a positive integer!');
end

f(1) = 1;
f(2) = 1;
for ii = 3:n
f(ii) = f(ii-2) + f(ii-1);
end
用编程loop实现A.*A的implicit loop:
edit mul
[row col] = size(A);
for r = 1:row
fprintf('Working on row %d...\n',r);
for c = 1:col
P(r,c) = A(r,c) * A(r,c);
fprintf('(%d %d)\n',r,c);
end
end
再一个例子：
edit asterisks
N = 7;
for mm = 1:N
for nn = 1:mm
fprintf('*');
end
fprintf('\n');
end
输出7行，分别是1~7个*
```

**6.2 While-Loops**

for-loops work well when we know the number of necessary iterations before entering the loop
Consider this problem:
Starting from 1, how many consecutive positive integers do we need to add together to exceed 50?
The only way to solve this with a for-loop is to guess a large enough number for the number of iterations and then use a break statement.
There is a better solution: a while-loop!
Difference between while & if:
while condition is evaluated repeatedly
block is executed repeatedly as long as condition is true

例子：
edit possum
function [n total] = possum(limit)
total = 0;
n = 0;
while total <= limit
n = n + 1;
total = total + n;
end
fprintf('sum: %d count: %d\n',total,n);
edit approx_sqrt
function y = approx_sqrt(x)
y = x;
while abs(y^2 - x) > 0.001*x
y = (x/y + y)/2;
end
如果另x为负数，则无法输出结果，将一直在计算，无限循环
i在matlab里是自定变量，可以给i赋值，只要clear i以后就可以回复原值
```


