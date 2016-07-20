---
layout: post
title: Smarty模板技术学习(二)
date: 2016-7-20
categories: blog
tags: [PHP]
description: PHP
---

**本文主要包括以下内容** 

1. 公共文件引入与继承  


#### 公共文件引入与继承        

可以把许多模板页面都用到的公共页面放到单独文件里边，通过模板就可以直接调用，类似php里边通过include指令引入公共文件一样。                   
{include  file=”[模目录/]板文件名称”}               

使用include对公共文件进行包含的好处是使用较简单，好理解。不好的地方使用相对较繁琐。     
许多框架程序把头部和脚部集中处理的技术成为“布局”       
在Smarty模板里边还可以通过“extends继承”对头部和脚部进行处理。       
extends相对include 代码也少一点、文件也相对较少，再者extends可以进行布局设计。             
 
{extends  file=”布局文件路径名”}          
{block name=”名称”}{/block}         

布局继承使用：         
①通过block标签在布局页面进行布局设计      
②在布局页面可以有两部分内容：html标签+block标签             
③在继承页面里边有两部分内容：extends + block，除此之外的内容不给显示        
④在继承页面通过{block name=”XXX”}来实现父级标签内容         
⑤在子级标签可以调用父级标签默认内容  {$smarty.block.parent} 或{$smarty.block.child}      
⑥父级block有默认内容，如果子级不实现会默认显示。如果实现则会覆盖。          

