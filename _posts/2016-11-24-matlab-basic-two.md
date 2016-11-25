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
