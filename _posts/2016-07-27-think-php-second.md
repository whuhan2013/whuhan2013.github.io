---
layout: post
title: ThinkPHP入门(二)
date: 2016-7-27
categories: blog
tags: [PHP]
description: PHP
---


#### smarty使用               
**smarty引入流程**             

```
1. 控制器IndexAction.class.php
    function index()
        $this -> display(); (父类Action的display)
2. 父类ThinkPHP/Lib/Core/Action.class.php
    function display()
        $this->view->display
3. ThinkPHP/Lib/Core/View.class.php
    function display()
        $this->fetch()
    function fetch()
        tag('view_parse',$params);
            ThinkPHP/Conf/tags.php
            view_parse => parseTemplate（Behavior行为）
4. 行为ThinkPHP/Lib/Behavior/parseTempateBehavior.class.php
    function run()
        $class = "TemplateSmarty";
        $tpl = new $class
        $tpl -> fetch()
5. TemplateSmarty.class.php
    ThinkPHP/Extend/Driver/Template/TemplateSmarty.class.php
    function fetch()
        vendor('Smarty.Smarty#class');
        //ThinkPHP/Extend/Vendor/Smarty/Smarty.class.php
        获取真正的smarty
        new Smarty();
        C（）函数会读取配置变量信息(convertion.php  config.php)
```

2.在config.php里边修改smarty的参数信息            
3.smarty布局继承效果          
布局继承  extends  block               

```
{extends file="public/layout.html"}
{block name="main"}
```                  

4.模板包含                 
{include file="public/ucenterleft.html"}              


5.display()显示具体模板           
$smarty -> display(模板名称);              
ThinkPHP框架调用模板：                  
① $this -> display();  tp框架会自动把摸板名称给拼装好，与操作名一致        
② $this -> display(模板名);   调用当前模块下的指定模板，模板没有后缀名          
③ $this -> display(模块/模板名);  调用其他模块下额指定模板           
④ $this -> display(相对路径模板);  了解           

#### 引入机制import                
include()   require         
通过import引入对应的类文件                
import(“hello.world.apple”);    hello/world/apple.class.php            
1.都可以引入什么地方的类文件                 
a)本项目的类文件可以引入  import(“@.dir.dir.file”);             
i.对应的类文件都需要创建在shop/Lib/XXX目录下边           
b)ThinkPHP核心类文件可以引入  import(“think.dir.dir.file”)        
i.对应类文件都设置在ThinkPHP/Lib/XXX                 
c)ThinkPHP/Extend 扩展库类文件可以引入  import(“ORG.dir.dir.file”);       
i.对应的类文件在ThinkPHP/Extend/Library/ORG/XXX                
d)特殊类引入，#号使用          

#### 登陆功能
**产生验证码**           

```
 //生成验证码
    function verifyImg(){
        import("ORG.Util.Image");
        echo Image::buildImageVerify();
    }
```

**session操作**     

```
 //持久化用户信息(id和名字)
session("mg_name",$user_info['mg_name']);
session("mg_id",$user_info['mg_id']);
```


**分页实现** 

```
function showlist(){
        $goods_model = D("Goods");
        //1 引入分页类
        import("@.components.Page");
        
        //2 计算当前记录总数目
        $total = $goods_model -> count();
        $per = 5;
        
        //3. 实例化分页类对象
        $page = new Page($total,$per);
        
        //4. 制作一条sql语句获得每页信息
        $sql = "select * from sw_goods ".$page->limit;
        $info = $goods_model -> query($sql);
        
        //5. 获得页面列表
        $page_list = $page->fpage(array(3,4,5,6,7,8));
        
        $this -> assign('info',$info);
        $this -> assign('page_list',$page_list);
        $this -> display();
    }
```

#### 缓存使用           
缓存：把数据库中的信息获取出来，放到一个缓冲介质里边，在相当一段时间之内，重复的数据就去缓存里边读取。

缓存介质：内存、file文件、数据库

不同的缓存介质，操作的方式不一样              

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p8.png)

具体使用

```
 function s1(){
        //缓存设置
        //缓存时间默认是永久的，可以设置
        S("username","linken");
        S("age",25);
        S("address","北京".time(),10); //过期自动删除
        S("goods_info",array('one'=>'apple','two'=>'htc','three'=>'nokia'));
        echo "ok ,success";
    }
    function s2(){
        //读取缓存信息
        echo S('username')."<br />";
        echo S('age')."<br />";
        echo S('address')."<br />";
        print_r(S("goods_info"));
    }
```

缓存案例

```
 //获取商品信息
    function getInfo(){
        //1 首先去缓存里边获得商品信息
        $goods = S("info");
        //2. 如果缓存里边有商品信息，直接返回,
        //   否则去数据库获得数据,并生成缓存供下次调用
        if(!empty($goods)){
            return $goods;
        } else {
            $goods = "apple".time();  //从数据获得商品信息
            //再把信息放入缓存，供下次调用
            S("info",$goods,10);
            return $goods;
        }
    }
```


#### 多语言设置      
1.进行多语言配置config.php            

```
 //配置多语言参数
    'LANG_SWITCH_ON'        => true,   // 默认关闭语言包功能
    'LANG_AUTO_DETECT'      => true,   // 自动侦测语言 开启多语言功能后有效
    'LANG_LIST'             => 'zh-cn,zh-tw,en-us', // 允许切换的语言列表 用逗号分隔
    'VAR_LANGUAGE'          => 'hl',        // 默认语言切换变量
```

2.配置行为Behavior执行      

```
<?php

return array(
    'app_begin'     =>  array(
        //以下行为会一次执行，自动加载机制会引入对应的文件
        'ReadHtmlCache','CheckLang' // 读取静态缓存
    ),
);
```

3.具体语言文件设置：           
4.具体语言使用             
$this -> assign('language',L());



#### 自动完成        

收集表单信息，把数据存入数据库              
可以使用”自动完成”机制对即将入库的信息进行二次处理         
例如：密码加密、用户注册时间等等。         

自动完成 类似 表单验证           
表单验证在create()方法内部触发           
自动完成 也在create()方法内部触发                

```
 //自动完成处理
    // 自动完成定义
    protected $_auto            =   array(
        //array(填充字段,填充内容,[填充条件,附加规则])
        array('password','md5',3,'function'),
        array('user_time','time',1,'function'),
        //array('user_time','abc',1,'callback'),
        //array('user_time','user_qq',1,'field'),
        //array('user_time','123456789',1,'string'),
    );
```

**自动映射** 

```
/进行自动映射，把一个假的表单域名称 与 真实的数据表字段名称对应起来
    // 字段映射定义
    protected $_map             =   array(
        'email' => 'user_email',
        'qq'    => 'user_qq',
    );  
```


#### 面向切面编程          
程序开发、执行不同的环节、不同的功能利用不同的文件进行处理。
把一个大块的功能切割为小块进行开发、执行
