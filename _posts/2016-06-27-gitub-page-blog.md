---
layout: post
title: 利用githubpages创建你的个人博客
date: 2016-6-27
categories: blog
tags: [github]
description: 利用githubpages创建你的个人博客
---

最近好多人都开始创建自己的博客网站写博客了, 有钱的买域名买主机好好的折腾一番. 没钱的就使用githubpages搭建自己的博客, 使用githubpages只能放静态网页, 不过这难不倒那些开了挂的coder, 各种静态博客生成器应运而生, 例如比较出名了hexo,jkelly.最近hexo比较火，不过github支持对jeklly的直接解析。

**jekyll** 

- [如何搭建一个独立博客——简明 Github Pages与 jekyll 教程 - 读立写生](http://cnfeat.com/blog/2014/05/10/how-to-build-a-blog/)     
- [GitHub 博客简明使用指南 - Microdust](http://azeril.me/blog/Github-Pages-Blog.html)      
- [我的 Github 个人博客是怎样炼成的 - 简书](http://www.jianshu.com/p/4fd3cb0a11da)             


**hexo**   

- [利用githubpages创建你的个人博客 - Loader's Blog - 博客频道 - CSDN.NET](http://blog.csdn.net/qibin0506/article/details/51813428)          
- [最简便的方法搭建Hexo-Github博客-基于Next主题 - 简书](http://www.jianshu.com/p/5e9bd5e39ae6)        
- [5分钟教你使用 github pages 搭建博客 - 简书](http://www.jianshu.com/p/bb7f9dcf556b)           
- [如何搭建一个独立博客——简明Github Pages与Hexo教程 - 简书](http://www.jianshu.com/p/05289a4bc8b2)


**改进代码高亮风格为AndroidStudio默认风格**           

- 添加androidstudio.css与highlight.pack.js到相应文件夹      
- 修改head.html

```
 <!-- add highlight -->
    <link rel="stylesheet" href="{{ "/css/androidstudio.css" | prepend: site.baseurl }}">
        <script type="text/javascript" src="{{"/js/highlight.pack.js" | prepend:site.baseurl}}"></script>
        <script>hljs.initHighlightingOnLoad();</script>
```

- 注释footer.html中关于highlight的部分**           

各种高亮效果预览:[https://highlightjs.org/static/demo/](https://highlightjs.org/static/demo/)           

**参考**          
[采用Jekyll + github + pygments构建个人博客的最终说明 - 简书](http://www.jianshu.com/p/609e1197754c)          
[Windows下搭建本地Jekyll - Gityuan博客](http://gityuan.com/2015/06/07/build-jekyll/)

**Jekyll博客目录结构**      

- 文件夹_layouts：用于存放模板的文件夹，
- 文件夹_posts：用于存放博客文章的文件夹
- 文件夹css：存放博客所用css的文件夹,比如主题文件以及高亮文件都是放在此处的
- .gitignore：可以删掉，后面会将项目添加到git项目，所以这个不需要了
- _coinfig.yml：jekyll的配置文件，里面可以定义相当多的配置参数，具体配置参数可以参照其官网
- index.html：项目的首页
- _includes:用于存放一些固定的HTML代码段，文件为.html格式，可以在模板中通过liquid标签引入，常用来在各个模板中复用如 - 导航条、标签栏、侧边栏之类的在每个页面上都一样不变的内容，需要注意的是，这个代码段也可以是未被编译的，也就是说也可- 以使用liquid标签放在这些代码段中
- 通过修改配置文件_coinfig.yml以及post和layouts就可以实现主要blog的建立和修改。



