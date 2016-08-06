---
layout: post
title: python基础语法
date: 2016-8-6
categories: blog
tags: [python]
description: python基础语法.
---


python基础语法，主要基于2.7，对于3.0的改动有部分指出    

链接：[几分钟快速入门Python - 简书](http://www.jianshu.com/p/a155eb855e18/comments/3411556#comment-3411556)


### python基础语法            

**输入输出**              

```
用print()在括号中加上字符串，就可以向屏幕上输出指定的文字。比如输出'hello, world'，用代码实现如下：
>>> print('hello, world')
print()函数也可以接受多个字符串，用逗号“,”隔开，就可以连成一串输出：

>>> print('The quick brown fox', 'jumps over', 'the lazy dog')
The quick brown fox jumps over the lazy dog
print()会依次打印每个字符串，遇到逗号“,”会输出一个空格
```


**输入**

```
现在，你已经可以用print()输出你想要的结果了。但是，如果要让用户从电脑输入一些字符怎么办？Python提供了一个input()，可以让用户输入字符串，并存放到一个变量里。比如输入用户的名字：

>>> name = input()
Michael
当你输入name = input()并按下回车后，Python交互式命令行就在等待你的输入了。这时，你可以输入任意字符，然后按回车后完成输入。

输入完成后，不会有任何提示，Python交互式命令行又回到>>>状态了。那我们刚才输入的内容到哪去了？答案是存放到name变量里了。可以直接输入name查看变量内容：

>>> name
'Michael'
```

