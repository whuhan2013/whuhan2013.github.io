---
layout: post
title: Makefile学习
date: 2017-1-9
categories: blog
tags: [linux]
description: Makefile学习
---

**makefile及make简介**      
在介绍makefile和make的具体概念前，我们先通过一个例子来说明makefile和make到底是为解决什么问题而存在的。        

假设有一个如图所示的C工程：    
![](https://dn-anything-about-doc.qbox.me/document-uid298389labid2374timestamp1481542407132.png/wm)  

那么要编译出我们的可执行程序project_demo，必须执行以下命令：

```
# 步骤1：编译主程序模块
$ gcc -o <100个主程序模块的o文件> -c <100个主程序模块的c文件>

# 步骤2：编译功能模块a
$ gcc -o <1000个功能模块a的o文件> <1000个功能模块a的c文件>
$ ar rcs liba.o <1000个功能模块a的o文件>

# 步骤3：编译功能模块b
$ gcc -o <1000个功能模块b的o文件> <1000个功能模块b的c文件>
$ ar rcs libb.a <1000个功能模块b的o文件>

# 步骤4：生成可执行文件demo
$ gcc -o demo <100个主程序模块的o文件> -L. -la -lb
```

上述例子反映了两个问题：       
1.上述4个编译步骤中，几乎每个步骤都有冗长的文件名列表需要输入而且有些还是重复的，这种工作枯燥而又费时，且极易因人为疏忽而出现错误；    
2.假设我们的demo项目每次编译所需的时间都比较长，那么如果我们之前已经成功编译过了我们的demo项目，而此后当我们修改了某些源文件需要更新的demo文件时，理论上我们是不希望也不需要去重新编译整个项目的，我们只需要仅分析其中的依赖关系，仅执行需要重新编译链接的命令，以节省编译时间，但是对于依赖关系非常复杂的工程而言，分析源文件涉及到的依赖关系是个非常复杂且容易出错的过程。         

make和makefile的存在正是为了解决上述两个问题的：

1.makefile文件帮助我们记录了整个项目工程的所有需要编译的文件列表，这样我们在编译时仅需要输入简单的make命令就能编译出我们期望的结果     
2.makefile文件反映了整个项目中各个模块的依赖关系，这样我们改动了某些源文件后，仅需简单的输入make命令，make工具就会根据makefile文件里描述的依赖关系帮助我们分析哪些模块需要重新编译，并执行相应的操作。          

在linux/unix开发环境中，makefile文件则是描述了一个特定编译系统所需要的策略，而make工具则是通过解析makefile文件并执行相应的命令来帮助我们构建其编译系统。

下面我们就带着这样两个问题来认识makefile和make工具：

1.makefile如何记录整个项目工程的所有需要编译的文件列表及如何反映整个项目中各个模块的依赖关系？       
2.提供了makefile策略描述后，make工具又是是如何解析makefile文件来帮助我们构建其编译系统的？    

**makefile简介**        
makefile就是一个简单的文本文件，它基本上就是由一条条的规则构成。下面，我们就来看一下makefile里的最基本的语法单元，规则。       
一条makefile的规则构成如下          

```
target:prerequisites
<tab> command1
<tab> command2
.....
<tab> commandN
```       

- target：规则的目标，可以简单理解为这条规则存在的目的是什么。通常是程序中间或者最后需要生成的文件名，也可以不对应具体的文件，而仅仅就是个概念上的规则目标。
- prerequisites：规则的依赖列表，可以简单的理解为要达到本条规则的目标所需要的先决条件是什么。可以是文件名，也可以是其他规则的目标；
- command：规则的命令，可以简单的理解为当目标所需要的先决条件的满足了之后，需要执行什么动作来达成规则的目标。规则的命令其实就是shell命令。一条规则中可以有多行命令，特别注意：每行命令都必须以tab键开始！            

**make命令工作机理**   
解释make的工作机理，需分别回答以下3个问题：

1.make命令如何使用 ；      
2.make从哪读取makefile；        
3.make如何解析执行makefile文件的规则。        

make命令的基本使用范式如下：

make [ -f makefile ] [ options ] ... [ targets ] ...         
使用make命令的最简单的方式主要有以下四种形式：         

```
1.简单粗暴，不带任何参数，直接执行make：
$ make
2.指定makefile文件：
$ make -f <makefile_name>
3.指定makefile 目标：
$ make <target>
4.到指定目录下执行make：
$ make -C <subdir> <target>
```

在执行make的时候，我们可以带上-f <文件名>参数，来指定make命令从哪里读取makefile文件；而如果我们不显式指定，则make就会在当前目录下依次查找名字为GNUmakefile, makefile,和 Makefile的文件来作为其makefile文件。       

在读取完makefile的内容后，make工具并不是逐条去执行makefile里的规则，而是以某条规则为突破口，多米诺骨牌效应式的去执行makefile里的规则。而这条作为突破口的规则的目标，称为终极目标 ，我们可以在执行make时以参数的形式指定终极目标，从而执行作为突破口的规则，如果我们不显式指定终极目标，make一般情况下将选择makefile的第一条规则的目标作为终极目标。

一般情况下，make执行一条规则的具体过程是这样的：                
![](https://dn-anything-about-doc.qbox.me/document-uid298389labid2374timestamp1481542499635.png/wm)      

**通过实例理解makefile的基本概念**        
本节我们将通过构建一个简单的c语言项目工程(我们命名为project_simple)来理解makefile的基本概念。

先来看一下project_simple的整体目录结构：        
![](https://dn-anything-about-doc.qbox.me/document-uid298389labid2374timestamp1481542575689.png/wm)     
因此我们的makefile可以这么写：

```
simple: main.c simple.c
    gcc -o simple main.c simple.c
```

**丰富我们的makefile语法工具箱**      
在第二章中，我们了解到makefile中最主要的语法单元是规则。本章我们将学习makefile中为了帮助我们更高效的编写出更加实用灵活的makefile而存在的帮助性语法。

以下是一般情况下一个完整的makefile所包含的语法模块：   
![](https://dn-anything-about-doc.qbox.me/document-uid298389labid2374timestamp1481542688710.png/wm)

**complicated项目构建**       
我们仿照project_simple项目，构建一个名为project_complicated的项目工程，目录结构如下：
![](https://dn-anything-about-doc.qbox.me/document-uid298389labid2374timestamp1481542725973.png/wm)     
代码内容:

```
// ---------------------------------------------
// main.c
#include <stdio.h>
#include "complicated.h"
int main()
{
    printf("%s\n", HELLO_STRING);
    complicated();
    return 0;
}
// ---------------------------------------------
// complicated.h
#ifndef __COMPLICATED_H__
#define __COMPLICATED_H__
#define HELLO_STRING "Hello !"
#define PROJECT_NAME "complicated"
extern void complicated(void);
#endif
// ---------------------------------------------
// complicated.c
#include <stdio.h>
#include "complicated.h"
void complicated(void)
{
    printf("This is a %s porject!\n", PROJECT_NAME);
}
```

项目的依赖关系：       
![](https://dn-anything-about-doc.qbox.me/document-uid298389labid2374timestamp1481542738231.png/wm)      
对比simple项目，我们的complicated项目貌似只是多了一个头文件，但是我们的依赖关系图却多了一层.o文件，这是为何？熟悉gcc编译过程的朋友应该知道，其实我们在用gcc 编译出可执行文件的过程中是包含两个阶段的：编译阶段和链接阶段。我们上述的依赖关系图更加准确的反映出了整个项目的构建过程，这样我们据此写出来的makefile才能更加灵活及更具可扩展性，记住：精确的分析清楚项目的依赖关系，是编写一个好的makefile的关键。

至此，我们也就可以轻易的写出complicated项目的makefile了：

```
complicated: main.o complicated.o
    gcc -o complicated main.o complicated.o

main.o: main.c
    gcc -o main.o -c main.c

complicated.o: complicated.c
    gcc -o complicated.o -c complicated.c
```

**参考：**[跟我一起来玩转Makefile](https://whuhan2013.github.io/shiyalou/%E8%B7%9F%E6%88%91%E4%B8%80%E8%B5%B7%E6%9D%A5%E7%8E%A9%E8%BD%ACMakefile%20-%20%E5%AE%9E%E9%AA%8C%E6%A5%BC.htm)