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

**简历**  

[程序员必备技能：在Github Pages上部署自己的简历](https://zhuanlan.zhihu.com/p/22250197)


**改进代码高亮风格为AndroidStudio默认风格**           

- 添加androidstudio.css与highlight.pack.js到相应文件夹      
- 修改head.html

```
 <!-- add highlight -->
    <link rel="stylesheet" href="{{ "/css/androidstudio.css" | prepend: site.baseurl }}">
        <script type="text/javascript" src="{{"/js/highlight.pack.js" | prepend:site.baseurl}}"></script>
        <script>hljs.initHighlightingOnLoad();</script>
```

- 注释footer.html中关于highlight的部分       

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



### 添加latex解析功能           

**使用 MathJax 公式显示 JS 引擎**

JS 动态加载，解析速度有些慢。GitHub Pages 支持的 Kramdown 解析器默认使用该引擎解析渲染数学公式，使用时需要在页面引入 MathJax 库。公式书写语法参考 [Math Blocks](http://kramdown.gettalong.org/syntax.html#math-blocks)。
但是即便不改换 Markdown 解析器，只要加载 MathJax 库，仍然可以正确显示公式，博主亲测！查阅 Jekyll 内置的 Kramdown 代码也没看到有相关的配置，只是简单的将文本中的公式区块解析成 HTML 标签 ,最终的公式渲染工作还是由 JS 实现。

按照下面的步骤，可以在 Markdown 文本中书写数学公式：



(1) 安装 kramdown 包

```
gem install kramdown
```

(2) 在 _config.yml 中指定 Markdown 解析器

```
# Conversion
markdown: kramdown

# Markdown Processors
kramdown:    # Better to turn on recognition of Github Flavored Markdown
  input: GFM
```

(3) 再把下面的代码插入到 <head> 标签里
（如果你使用 Octopress，那就是添加到 /source/_includes/custom/head.html 文件里）

```
<!-- mathjax config similar to math.stackexchange -->

<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>

<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
      }
    });
</script>

<script type="text/x-mathjax-config">
    MathJax.Hub.Queue(function() {
        var all = MathJax.Hub.getAllJax(), i;
        for(i=0; i < all.length; i += 1) {
            all[i].SourceElement().parentNode.className += ' has-jax';
        }
    });
</script>

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

(4) 最后在 Markdown 文件里写公式代码         
例如，下面的 Cauchy-Schwarz Inequality：

```
$$
\left( \sum_{k=1}^n a_k b_k \right)^{\!\!2} 
\leq 
\left( \sum_{k=1}^n a_k^2 \right) 
\left( \sum_{k=1}^n b_k^2 \right)
$$
```

就会得到：

$$
\left( \sum_{k=1}^n a_k b_k \right)^{\!\!2} 
\leq 
\left( \sum_{k=1}^n a_k^2 \right) 
\left( \sum_{k=1}^n b_k^2 \right)
$$

```
而段内插入 LaTeX 公式是这样的： $ \{\,z\in C \mid z^2 = {\alpha}\,\} $，试试看看吧
```

可以得到：

而段内插入 LaTeX 公式是这样的： $ \{\,z\in C \mid z^2 = {\alpha}\,\} $，试试看看吧


**参考链接：**[GitHub Pages 静态博客 - 个人建站实录](http://alfred-sun.github.io/blog/2014/12/05/github-pages/)