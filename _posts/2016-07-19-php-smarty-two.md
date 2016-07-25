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
2. 内容捕捉     
3. 变量调剂器       
4. 缓存    
5. Smarty过滤器         
6. 数据对象、注册对象           
7. 与已有项目结合

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


#### 内容捕捉        

```
<body>
        <h2>最新上架商品信息</h2>
        {*
        在smarty许多标签里边都有assign属性
        作用：把标签包含的内容都赋予assign变量里边
              像include标签也有该属性，
              有assign属性的标签内容，不给立即显示，必须通过变量输出。
        *}
        {capture name="gds" assign="shangpin"}
        {foreach $gd as $k => $v}
            {$v@iteration}---{$v}<br />
        {/foreach}
        {/capture}
        {$smarty.capture.gds}

        <h2>用户非常喜欢的商品信息</h2>
        {$smarty.capture.gds}

        <h2>本月最畅销商品信息</h2>
        {$shangpin}
    </body>
```

#### 变量调剂器       
将变量按照我们所需要的格式输出


```
 <body>
        <h2>变量调剂器</h2>
        {$country}<br />
        {$country|upper}<br />
        {$country|lower}<br />
        {$country|capitalize}{*首字母大写*}<br />
        {$country|cat:'beijing':'西三旗'}{*链接字符串*}<br />
        {$smarty.now}<br />
        {$smarty.now|date_format:"%Y-%m-%d %H:%M:%S"}<br />
        {$content}<br />
        {$content|escape}{*把html标签的<>转为符号实体*}<br />
        {$content|replace:'this':'that'|escape}{*对单词进行替换,并且递归使用第二个escape调剂器*}<br />
        {$introduce}<br />
        {$introduce|truncate:22:'..':true}{*默认给按照单词截取*}<br />

    </body>
```

#### 缓存         

缓存分为：页面、数据             
页面缓存：把php解释器解释好的内容给放到一个单独文件里边，这个单独文件可以被反复调用。          
模板文件、编译文件、缓存文件              
缓存文件：编译文件经过php模块编译运行生成的静态内容形成的文件就是缓存文件。             


**缓存具体设置：**           
①设置缓存文件目录cache                          
②开启缓存设置：  $smarty -> caching = true;                          
③调用方法: $smarty -> display(); 该display方法会判断caching属性是否开启，若开启会把具体的“缓存文件”给自动创建好   

**display()方法**              
该方法的作用            
1.加载模板文件                               
2.判断是否开启缓存和是否有缓存文件，如果都符合，则直接调用缓存文件返回给用户结果。        
如果有开启缓存，则会生成缓存文件；缓存文件已经过期，也会重新给生成。                 
3.判断是否有对应的编译文件，有则直接加载，通过php模块解释返回。                         
4.如果没有开启缓存、也没有对应编译文件，则对模板文件进行编译，再把编译文件给php模块解释，最后返回。    

**缓存文件更新**               
①对应的模板文件发生更新操作         
②可以给缓存文件设置过期时间                
$smarty -> cache_lifetime = 3600;              


**缓存有效期判断有两个：**         
①Smarty的成员属性判断cache_lifetime。         
②可以根据缓存文件本身的有效期时间进行判断。         

**缓存caching设置**                
① $smarty -> caching = 1或true;   该情况有效期判断根据Smarty成员属性cache_lifetime判断        
② $smarty -> caching = 2;  该情况有效期判断根据 缓存文件本身有效期时间的有效判断。           


**单模板多缓存**            

**缓存文件删除**                                    
clearCache(模板)  删除指定模板对应的全部缓存文件              
clearCache(模板，标志)  删除当前模板指定标志的缓存文件            
clearAllCache()  删除全部的缓存文件           

**局部不缓存4种方法**                 
① {$变量名称  nocache}   {$title  nocache}      
② $smarty -> assign(name,value,true);     
③ {nocache}内部内容都不给缓存foreach{/nocache}           
④ insert  函数不缓存设置              


**缓存集合**        
缓存集合：是对单模板多缓存升级的一种缓存效果          
http://wangzhan/goods.php?brand=samsung&price=3&network=unicom&screen=4&color=black         
缓存集合：要把商品全部的参数做“排序组合”，每种情况都设置一个缓存文件             

**与MVC结合使用**        
①在入口文件引入Smarty          
②在父类控制器里边实例化Smarty对象，通过父类属性接收该对象         
③在子类里边调用父类的smarty属性，进行smarty相关操作。             

**缓存文件制作**      

```
ob_get_contents()  抓取php缓存区内容
                    注意：抓取该函数以上行的内容
php缓冲区开关
① ob_start();    开启php缓冲区
②  php.ini 
    output_buffering = 4096 开启
    output_buffering = Off  关闭

ob_clean()      清除php缓冲区内容
                清除当前行以上的缓冲区内容，以下行的内容仍然可以抓取使用

ob_get_clean()  抓取缓冲区内容，并清除之
                抓取/删除 当前行以上的信息，以下行信息没有影响
```


#### 过滤器

```
<?php

include "./MySmarty.class.php";

$smarty = new MySmarty;

$smarty -> caching = 1;

//① 前置过滤器pre

//$smarty -> registerFilter("pre",处理函数);
$smarty -> registerFilter("pre","beforeC");

//$tpl是"模板文件"内容
function beforeC($tpl){
    //删除$tpl的注释内容
    return preg_replace("/<!--.*-->/","",$tpl);
}
/*
    $mark: 表示过滤器类型(前置、后置、输出)
    function registerFilter($mark,$callback){
        $cont = file_get_contents(模板文件名称);//获得模板内容
        $callback($cont);
    }
*/

//② 后置过滤器post
//   给混编文件统一设置作者信息
$smarty -> registerFilter("post","afterC");
//$tpl 是编译后的混编代码内容
function afterC($tpl){
    return "<!--create by itcast0421-->".$tpl;
}

//③ 输出过滤器output
//   对关键字进行过滤
$smarty -> registerFilter("output","outC");
//是php模块解释后的静态内容
function outC($tpl){
    return str_replace('2014-06-14','date20140614',$tpl);
}

$smarty -> assign('qi','2014-06-14');

$smarty -> clearCompiledTemplate('03.html');//清除旧的编译文件

$smarty -> display('03.html');
```


#### 数据对象、注册对象        

**数据对象**

```
<?php

include "./MySmarty.class.php";

$smarty = new MySmarty;

//有时候，页面内容非常多，这样不同程序员开发自己的php代码，
//调用自己的一部分模板

//由于页面非常大，彼此定义的变量名称"一致"

//以下变量其实是最后一个会起作用
//解决：
//    ① 把变量名称给改一下
//       不推荐，代码高耦合，一个地方修改另一个地方也要修改
//    ② 把页面调用调到前面
//       不推荐，display代码受到位置限制
//    ③ 利用数据对象实现

//三个盒子(数据对象)
$data1 = $smarty -> createData();
$data2 = $smarty -> createData();
$data3 = $smarty -> createData();

//通过数据对象$data1  $data2  $data3为各自模板传递变量信息
$title = "乌克兰发送内战";
$data1 -> assign('title', $title); //为当前盒子填充变量信息
$data1 -> assign('age',20);

$title = "京津冀一体化";
$data2 -> assign('title', $title);//为当前盒子填充变量信息
$data2 -> assign('age',21);

$title = "巴西世界杯";
$data3 -> assign('title', $title);//为当前盒子填充变量信息
$data3 -> assign('age',22);

$smarty -> display("041.html", $data1); //在自己盒子获得变量信息
$smarty -> display("042.html", $data2);
$smarty -> display("043.html", $data3);
```

**注册对象**          
在php里边可以给模板传递一个对象变量，对象可以通过”->”调用自己的属性和方法。       
注册对象：可以限制对象在模板中调用什么方法/属性。      

```
<?php

include "./MySmarty.class.php";

$smarty = new MySmarty;

//注册对象
class Order{
    public $type = "电子产品";

    //$args是当前函数的形式参数，会接收模板传递过来的全部信息
    function getInfo($args){
        //从数据库获得订单信息
        return $args['name']."--".$args['number']."--1个T桖、3双袜子";
    }

    function getPrice(){
        //从数据库获得价格信息
        return 59;
    }

    function getKou(){
        return 7.5;
    }
}
$dingdan = new Order;
//注册对象具体使用
//$smarty -> registerObject(模板变量，对象，允许被调用的"属性/方法"的数组);
$smarty -> registerObject('dingdan', $dingdan, array('getInfo','getPrice','type'));
$smarty -> display("05.html");
```


#### 与已有项目结合       
引入smarty后，auto_load自动加载失效。                           
Smarty与已有框架结合重点：把加载机制级别设置与Smarty一致。       

spl_autoload_register声明会覆盖原始的autoload加载机制，引入全部的加载机制都声明为spl_autoload_register就可以分别执行。    

spl_autoload_register("__autoload");







