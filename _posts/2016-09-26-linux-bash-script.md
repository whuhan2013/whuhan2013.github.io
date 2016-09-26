---
layout: post
title: Bash脚本编程
date: 2016-9-26
categories: blog
tags: [linux]
description: bash脚本
---

**什么是Bash?**

1.bash简介

Bash（GNU Bourne-Again Shell）是一个为GNU计划编写的Unix shell，它是许多Linux平台默认使用的shell。

shell是一个命令i解释器，是介于操作系统内核与用户之间的一个绝缘层。准确地说，它也是能力很强的计算机语言，被称为解释性语言或脚本语言。它可以通过将系统调用、公共程序、工具和编译过的二进制程序”粘合“在一起来建立应用，这是大多数脚本语言的共同特征，所以有时候脚本语言又叫做“胶水语言”
事实上，所有的UNIX命令和工具再加上公共程序，对于shell脚本来说，都是可调用的。Shell脚本对于管理系统任务和其它的重复工作的例程来说，表现的非常好，根本不需要那些华而不实的成熟紧凑的编译型程序语言。

#### 初步练习

1.Hello World

Bash之Hello World

```
$ vim hello.sh
使用vim编辑hello.sh ，输入如下代码并保存：

 #!/bin/bash
 # This is a comment
 echo Hello World
运行Bash脚本的方式：

# 使用shell来执行
$ sh hello.sh
# 使用bash来执行
$ bash hello.sh
还可以让脚本本身就具有可执行权限，通过chmod命令可以修改：

# 赋予脚本的所有者该执行权限，允许该用户执行该脚本
$ chmod u+rx hello.sh
# 执行命令，这将使用脚本第一行指定的shell来执行，如果指定shell不存在，将使用系统默认shell来执行
$  ./hello.sh
```

利用bash编程可以有效的提高工作效率，避免重复输入命令，比如上传github的bash脚本

```
cd ~/data/whuhan2013.github.io/
git add .
git commit -m 'update'
git push origin master
```



