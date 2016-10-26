---
layout: post
title: 基于Gitbook制作电子书
date: 2016-10-26
categories: blog
tags: [github]
description: 基于Gitbook制作电子书
---


**本文主要包括以下内容**         

- Markdown 书写
- Git 使用
- Gitbook 基本操作
- Github pages


**效果如下**      

![](https://dn-anything-about-doc.qbox.me/document-uid53532labid2137timestamp1474437087328.png/wm)


** Gitbook 安装**

```
sudo npm install -g gitbook-cli
sudo npm install -g gitbook
安装完之后，你可以检验下是否安装成功。

gitbook -V
```

**编辑书籍** 

安装好 Gitbook 之后，我们就可以创建图书了。

Gitbook 的基本用法非常简单，基本上就只有两步:

1. 使用 gitbook init 初始化书籍目录        
2. 使用 gitbook serve 编译书籍      

首先，进入一个目录，例如之前我们创建好的 gitbook，执行初始化命令：

sudo gitbook init       
然后我们的 gitbook 空目录会多出两个文件：

```
gitbook/
├── README.md
└── SUMMARY.md
```

README.md 和 SUMMARY.md 是两个必须文件，README.md 是对书籍的简单介绍。SUMMARY.md 是书籍的目录结构。

对上面两个文件，我们做一下编辑：

**REAMDE.md:**

```
# Introduction
This is a book powered by [GitBook](https://github.com/GitbookIO/gitbook).
SUMMARY.md:
```

**Summary**

```
* [Introduction](README.md)
* [第一部分](chapter1/README.md)
    * [1.1 数学](chapter1/math/math.md)
        * [1.1 高等数学](chapter1/math/advance.md)
        * [1.2 离散数学](chapter1/math/lisan.md)
    * [1.2 编程语言](chapter1/program/program.md)
        * [1.2.1 C 语言](chapter1/program/c.md)
        * [1.2.2 C++](chapter1/program/c++.md)
        * [1.2.3 Java](chapter1/program/java.md)
    * [1.3 软件工程](chapter1/software/software.md)
    * [1.4 数据结构](chapter1/data_structure/data_structure.md)
    * [1.5 数据库原理](chapter1/database/database.md)
    * [1.6 计算机网络](chapter1/network/compute_network.md)
    * [1.7 计算机系统](chapter1/system/compute_system.md)
        * [1.7.1 操作系统](chapter1/system/operating.md)
        * [1.7.2 微机接口](chapter1/system/interface.md)
        * [1.7.3 计算机组成原理](chapter1/system/comprise.md)
        * [1.7.4 计算机系统结构](chapter1/system/system_structure.md)
* [第二部分](chapter2/README.md)
    * [2.1 算法](chapter2/algorithm/algorithm.md)
    * [2.2 数据挖掘](chapter2/data_mining/data_mining.md)
    * [2.3 机器学习](chapter2/learning/machine_learning.md)
    * [2.4 搜索引擎](chapter2/search/search_engine.md)
```

编辑这两个文件之后，再次执行：

gitbook init       
它会为我们创建 SUMMARY.md 中的目录结构。  

书籍目录结构创建完成以后，就可以使用 gitbook serve 来编译和预览书籍了

gitbook serve 命令实际上会首先调用 gitbook build 编译书籍，完成以后会打开一个 web 服务器，监听在本地的 4000 端口。

现在，可以用浏览器打开 http://localhost:4000 查看书籍的效果，如下图：


![](https://dn-anything-about-doc.qbox.me/document-uid53532labid2137timestamp1474440390552.png/wm)

现在，Gitbook 为我们创建了书籍目录结构后，就可以向其中添加真正的内容了，文件的编写使用 Markdown 语法，在文件修改过程中，每一次保存文件，gitbook serve 都会自动重新编译，所以可以持续通过浏览器来查看最新的书籍效果！



**通过gitbook.com管理书籍**        
略


#### 发布到 GitHub      

源代码保存到 master 分支，编译出来的静态文件 _book 上传到 gh-pages 分支，这样我们就可以通过 GitHub pages 来发布电子书了。

具体操作:

- 登录到Github，创建一个新的仓库，名称我们就命令为 book，这样我就就得到了一个 book 的空仓库;
- 克隆仓库到本地：git clone git@github.com:USER_NAME/book.git;
- 创建一个新分支：git checkout -b gh-pages，注意，分支名必须为 gh-pages;
- 将分支 push 到仓库：git push -u origin gh-pages;
- 切换到主分支: git checkout master。

经过这一步处理，我们已经创建好 gh-pages 分支了，有了这个分支，GitHub 会自动为你分配一个访问网址：http://USERNAME.github.io/book。

操作到这一步，我们所在目录是在 Code/book/ 下，现在我们需要在 Code 目录下保存我们的电子书的静态文件，切换到Code 目录，克隆远程仓库 book 的 gh-pages 分支并保存为 book_build:

```
git clone -b gh-pages git@github.com:USERNAME/book.git book-build
```

然后我们从 Code/book/_book 目录下把编译好的电子书静态文件复制到 Code/book_build 下，执行：

```
sudo git add .
sudo git commit -m "add e-book static file"
sudo git push origin gh-pages
```

将静态文件 push 到远程仓库 book 的 gh-pages 分支。

然后，等十来分钟的样子，你就可以通过访问网址：http://USERNAME.github.io/book来访问到你的在线图书了。之后，每次修改之后，都可以将生成的静态文件 copy 到 book-build 目录，再 push 到远程仓库 book 的 gh-pages 分支。

**参考链接**           

[基于 Gitbook 制作电子书](https://www.shiyanlou.com/courses/665/labs/2137/document)





