---
layout: post
title: JAVA求职专项练习(二)
date: 2016-7-27
categories: blog
tags: [求职]
description: 
---

**1、静态初始化代码块、构造代码块、构造方法**               

```
当涉及到继承时，按照如下顺序执行：
1、执行父类的静态代码块 
static {
        System.out.println("static A");
    }
输出:static A
2、执行子类的静态代码块
static {
        System.out.println("static B");
    }
输出:static B
3、执行父类的构造代码块
{
        System.out.println("I’m A class");
    }
输出:I'm A class
4、执行父类的构造函数
public HelloA() {
    }
输出：无
5、执行子类的构造代码块
{
        System.out.println("I’m B class");
    }
输出:I'm B class
6、执行子类的构造函数
public HelloB() {
    }
输出：无
```  


**2、关于PreparedStatement与Statement描述错误的是**         
