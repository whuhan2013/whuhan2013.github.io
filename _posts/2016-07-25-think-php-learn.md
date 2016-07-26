---
layout: post
title: ThinkPHP入门
date: 2016-7-25
categories: blog
tags: [PHP]
description: PHP
---

**ThinkPHP项目的创建**            

```
<?php
include "../../ThinkPHP/ThinkPHP.php";
```

在index.php中导入ThinkPHP.php即可，会自动创建目录。  


**tp创建流程分析**           

```
1. 入口文件index.php

2. ThinkPHP/ThinkPHP.php
    require THINK_PATH.'Common/runtime.php';

3. ThinkPHP/Common/runtime.php
    声明许多常量信息
    加载系统核心类文件
    if(is_file($file))  require_cache($file);
    自动创建应用目录
    build_app_dir();
    Think::Start();
4. ThinkPHP/lib/Core/Think.class.php
    static function start(){}
    Think::buildApp();         // 预编译项目
        引入相关配置文件
    App::run();
5. ThinkPHP/lib/Core/App.class.php
    static public function run(){}
    App::init();
        Dispatch分析路由
        分析路由(控制器MODULE_NAME  方法ACTION_NAME)  index.php?c=控制器&a=方法
    App::exec();
        通过反射ReflectionMethod使得控制器对象调用对应的方法
```


**PHP中的反射**  

```
<?php

class Person{
    public $name = "xiaoming";
    function say(){
        echo "I am ".$this->name;
    }

    function run($addr,$age){
        echo "I am ".$this->name;
        echo ",I at ".$addr;
        echo ", my age is ".$age;
        echo ",running";
    }
}
$per = new Person;
//$per -> say();

//利用反射实现对象调用方法
//$md = new ReflectionMethod(类名，方法名);
//反射方法对象
//反射好处：可以获得 方法的属性(是否公开的、私有的、受保护的等等)
//$md = new ReflectionMethod("Person","say");
//让指定的对象调用这个方法
//$md -> invoke($per);

//通过反射执行带参数的方法
$mda = new ReflectionMethod("Person","run");
$mda -> invokeArgs($per,array('beijing',23));
```


#### 空操作和空模块                   
tp框架把MVC中的控制器称作是模块(module)  index.php?m=控制器&a=操作     
http://网址/index.php/User/login  正常请求             
http://网址/index.php/User/beijing  “空操作”请求                   
http://网址/index.php/American/login  “空模块”            

1. 什么是空操作：         
	用户访问一个控制器中不存在的方法，就是空操作。
实例化对象，这个对象去调用类里边不存在的方法         
	在OOP中有魔术方法，__call()，自动调用方法，
对象调用类中不存在方法要统一走这个方法           

2.空操作处理：         
a)在对应的控制器里边定义方法_empty()               
b)给应用添加一个函数名字是：__hack_action() 推荐使用               
应用函数库文件：shop/Common/common.php               

3.空模块（空控制器）       
$tom = new  American();           
$tom是不存在的对象           

处理方式：       
①定义函数__hack_module()  推荐使用        
②给系统定义空模块：EmptyAction.class.php          


#### 跨模块调用
一个控制器可以实例化另一个控制器，并调用其相关方法           


```
/系统有自动加载机制，会完成控制器类的引入
        // ThinkPHP/Lib/Core/Think.class.php
        //$user = new UserAction();
        //echo $user -> number();
        
        //系统给我们提供快捷函数，实现控制器的实例化
        //A('控制器标志')
        //A('[项目://][分组/]控制器');
        //A() ThinkPHP/Common/common.php
        $user = A('home/User');
        echo $user -> number();
        $indx = A("book://Index");
        echo $indx -> apple();
        
        //R("[项目://][分组/]模块/操作方法")  跨模块调用函数
        //实例化控制器并直接调用相关方法
        //R()方法有封装A()方法，之后利用对象调用相关操作
        echo R("home/User/number");
        echo R("book://Index/apple",array('hello','world'));
```



#### ThinkPHP数据库操作             

在config.php中配置

```
 //数据库配置
    'DB_TYPE'               => 'mysql',     // 数据库类型
    'DB_HOST'               => 'localhost', // 服务器地址
    'DB_NAME'               => 'shop',          // 数据库名
    'DB_USER'               => 'root',      // 用户名
    'DB_PWD'                => '111111',          // 密码
    'DB_PORT'               => '',        // 端口
    'DB_PREFIX'             => 'sw_',    // 数据库表前缀
    'DB_FIELDTYPE_CHECK'    => false,       // 是否进行字段类型检查
    //
    //处于性能考虑，把数据表字段放入缓存里边，
    //这样下次访问就避免执行sql语句重复执行重新获取
    //开发调试模式APP_DEBUG=true,下边缓存无效
    //生产模式APP_DEBUG=false,缓存有效
    'DB_FIELDS_CACHE'       => true,        // 启用字段缓存
    
    //修改模板引擎为smarty
    'TMPL_ENGINE_TYPE'      =>  'Smarty',     // 默认模板引擎 
    
    //在页面底部显示日志信息
     'SHOW_PAGE_TRACE'   => true,   // 显示页面Trace信息
```

**实例化数据模型model的三种方法**            
1.$goods_model  = new GoodsModel();            
2.$goods_model  = D(“Goods”);   快捷函数             
a)$obj = D();  //创建了一个基类model的对象，没有指明具体操作的数据表            
b)通过这个$obj就可以执行原生的sql语句                 
3.$model = M();   创建基类model对象               
a)$model = M(‘User’);  创建基类model对象，但是操作的数据表sw_user                 


**查询数据信息**        

```
$goods_model = D('Goods');
$info=$goods_model->select();
```


