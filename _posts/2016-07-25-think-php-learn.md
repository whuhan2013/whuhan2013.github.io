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


#### 查询数据信息       
① Model.class.php类本身就存在该方法，例如(where()  field()  limit() select())           
② __call()自动调用方法集成了一些方法，例如(table()  order()   group())
    这些方法可以进行连贯操作$info = $obj -> where()->order()->limit()->select()

```
$goods_model = D('Goods');
$info=$goods_model->select();

//通过select（）方法查询数据
//返回二维数组
//select(记录主键值)
//select("13,26,33");
$info = $goods_model -> select();  全部记录信息
$info = $goods_model -> select(19);  //根据主键值查询指定记录
$info = $goods_model -> select("13,26,33");  //根据主键值查询若干条记录

//find()方法返回一个一维数组数据
//每次只返回一条记录信息
$info = $goods_model -> find(19);
$info = $goods_model -> find("29,32,45,19");
show_bug($info);

//限制字段查询field("字段,字段...");
$info = $goods_model -> select();  //全部字段、全部记录信息
$info = $goods_model->field("goods_name,goods_number,goods_create_time")->select();

//显示查询条数limit([偏移量，]长度)
$info = $goods_model -> limit(5)->select();
$info = $goods_model -> limit(5,5)->select();

//排序查询order("price asc/desc")
$info = $goods_model -> order('goods_price desc')->select();
$info = $goods_model -> order('goods_price desc')->limit(5)->select();
//model对象调用本身不存在的方法order()
//实际会执行魔术方法__call()自动调用

//下边的连贯操作，多个条件彼此没有顺序要求，最后都被传递给options属性
//options属性最后会把sql语句根据条件给拼装好
$info = $goods_model ->limit(5)->field('goods_name,goods_price')-> order('goods_price desc')->select();

//设置where条件 where()
$info = $goods_model->where('goods_price > 5000')->select();
$info = $goods_model ->table('sw_goods')->select();
$info = $goods_model -> group("goods_category_id")->select();
```


1.select()  返回二维数组         
2.find()  返回一维数组                           
3.getByXXX();  根据指定字段进行数据查询             
返回一维数组                     
getByGoods_price(12000);                   
getByGoods_name();    魔术__call();                      
4.having() 设置查询条件            
与where比较类似              
select  *  from  sw_goods  where  goods_price>100;          
select  *  from sw_goods  having  goods_price >100;              

```
function showlist(){
        $goods_model = D("Goods");
        //查询名字为"三星BC01"商品信息
        //$info = $goods_model -> getByGoods_name('三星BC01');
        //$info = $goods_model -> getByGoods_price('5999');
        //show_bug($info);
        //$info = $goods_model -> having("goods_name like '诺%'")->select();
        
        //执行原生sql语句
        $sql = "select * from sw_goods a join sw_category b on a.goods_category_id=b.cat_id";
        $info = $goods_model->query($sql);
        
        $this -> assign('info', $info);
        $this -> display();
    }
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p7.png)


**model聚合函数使用**      
count()   sum()   max()   avg()   min()


**执行原生sql语句**         
$model -> query()    查询语句   （二维数组返回）         
$model -> execute()  增加、修改、删除            
insert  update   delete             
成功执行返回受影响的记录数目           


#### 添加数据add
两种方式实现数据添加：数组方式、AR方式      
AR规则：            
1.数据库中的每个数据表都对应一个类,model        
2.数据表中的每条记录都是类的一个对象          
3.记录信息的每个字段都是对象的属性            
在tp框架中AR模型是假的      

数据查询有options属性汇集各种连贯查询条件       
数据添加有data属性实现各种信息的收集                 

```
//添加商品
    function add1(){
        $goods_model = D("Goods");
        //实现数据添加
        //数组下标 与 数据库表 字段 一致
        /*
        $d = array(
            'goods_name'=>'htc230',
            'goods_price'=>'3999',
            'goods_number'=>45,
            'goods_weight'=>103
        );
        $rst = $goods_model -> add($d);
        */
        
        //AR方式实现数据添加
        //对象调用不存在的属性需要调用魔术方法__set();
        /*
        $goods_model -> goods_name = "iphone6";
        $goods_model -> goods_price = "5700";
        $goods_model -> goods_number = 31;
        $goods_model -> goods_weight = 110;
         * 
         */
        //以上4个信息最后被$this->data收集
        $rst = $goods_model -> add();
        
        show_bug($rst);  //被添加记录本身的id值
        
        $this -> display();
    }
```

**通过表单form实现数据添加**           

```
    function add(){
        $goods_model = D("Goods");
        //判断是否提交表单
        if(!empty($_POST)){
            //数据添加
            //foreach($_POST as $k => $v){
            //    $goods_model -> $k = $v;
            //}
            $goods_model -> create();  //tp框架本身收集表单数据方法
            
            $rst = $goods_model -> add();
            if($rst){
                $this -> success('添加成功',__URL__."/showlist");
            }
        } else {
            $this -> display();
        }
    }
```

#### 修改数据       
查询：select()   添加：add()           
修改关键字：save()  返回受影响的记录数目              

添加数据：数组 和 AR             
修改数据：数组 和 AR              
注意：实现数据修改必须符合两个条件(主键id 或 where条件)               

```
 //修改商品
    function upd1(){
        $goods_model = D("Goods");
        //修改数据
        $d = array(
            'goods_name'=>'小米手机',
            'goods_price'=>3200
        );
        $rst = $goods_model->where('goods_id>50') -> save($d);
        //在实际生产中，是不允许一次性修改全部记录信息
        show_bug($rst);
        
//        $goods_model -> goods_name = "xxx";
//        $goods_model -> goods_price = "xxx";
//        $goods_model -> save();
        
        $this -> display();
    }
```

具体案例实现update                    
以前：http://网址/index.php?m=控制器&a=操作&goods_id=100&goods_price=2300          
现在：http://网址/index.php/控制器/操作/参数1/值/参数2/值/参数3/值。。。。。。            

```
function upd(参数1,参数2,参数3){
    //$_GET[‘参数1’];
    //$_GET[‘参数2’];
    参数1，参数2，参数3具体使用
}
```


```
 //以下方法被访问必须传递参数
    //http://网址/index.php/控制器/方法/goods_id/102/goods_price/305
    //url地址参数名字要求与方法参数一致
    function upd($goods_id,$goods_price=506){
        //把被修改的商品信息查询出来，传递给模板显示
        $goods_model = D("Goods");
        
        if(!empty($_POST)){
            //接收修改的表单数据
            $goods_model -> create(); //收集表单信息
            $rst = $goods_model -> save();
            if($rst){
                //$this -> success(提示信息，跳转地址);
                $this -> success('修改成功',__URL__."/showlist");
            }
        } else {
            $info = $goods_model -> getByGoods_id($goods_id);

            $this -> assign('info',$info);
            $this -> display();
        }
    }
```


#### tp框架中表单域验证        
表单验证：           
    通过tp框架进行表单验证            
    先决条件：收集表单数据必须通过create()收集        
    create()方法内部有集成表单验证规则           

**具体使用**  

```
//注册操作
    function register(){
        $user_model = D("User");
        if(!empty($_POST)){
            show_bug($_POST);
            $z = $user_model -> create();  //收集数据
            if($z){
                //把爱好的数组变为字符串
                $user_model -> user_hobby = implode(',',$_POST['user_hobby']);
                
                $rst = $user_model -> add();
                if($rst){
                    echo "ok";
                } else {
                    echo "failure";
                }
            } else {
                //表单域验证有错误
                show_bug($user_model -> getError());
            }

        }else {
        }
        $this -> display();
        
    }
```

在create()时会发生验证，验证的代码写在model中  

```
<?php

class UserModel extends Model{
    //批量错误信息显示
    // 是否批处理验证
    protected $patchValidate    =   true;
    
    //进行表单域验证
    //protected $_validate        =   array();  // 自动验证定义
    protected $_validate = array(
        //array(验证字段,验证规则,错误提示,[验证条件,附加规则,验证时间])
        array('username','require','用户名必须填写'),
        //密码必须填写
        array('password','require','密码必须填写'),
        //确认密码必须填写/与密码保存一致
        array('password2','require','确认密码必须填写'),
        array('password2','password','两次密码必须一致',0,'confirm'),
        //验证邮箱
        array('user_email','email','邮箱格式必须正确',2),
        //验证qq号码
        //全部数字、长度5-12位、第一位非0
        array('user_qq','/^[1-9]\d{4,11}$/','qq格式必须正确'),
        //手机号码验证，正则验证
        //学历必须选择一项
        //value值必须在一个范围内 2,3,4,5，
        array('user_xueli',array(2,3,4,5),'学历必须选择一项',0,'in'),
        //爱好，必须选择两个或以上项目
        //计算$_POST['user_hobby']这个数组里边元素的个数
        array('user_hobby','check_hobby','爱好必须选择两项以上',1,'callback'),
    );
    
    //验证爱好的方法
    function check_hobby($hobby){
        //获取$_POST['user_hobby']具体信息值
        //这个check_hobby被调用的时候已经把$_POST['user_hobby']
        //当成是参数给check_hobby了
        //show_bug($hobby);
        if(count($hobby) < 2){
            return false;
        } else {
            return true;
        }
    }
    
    //自定义个性方法进行用户名和密码校验
    function checkNamePassword(){
        //sljdlsjdlk
    }
}
```

第四个参数，验证条件      
验证条件：（可选）包含下面几种情况：       
Model::EXISTS_VALIDATE 或者0 存在字段就验证（默认）          
Model::MUST_VALIDATE 或者1 必须验证           
Model::VALUE_VALIDATE或者2 值不为空的时候验证         



