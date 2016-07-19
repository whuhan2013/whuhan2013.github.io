---
layout: post
title: Smarty模板技术学习
date: 2016-7-19
categories: blog
tags: [PHP]
description: PHP
---

模板引擎技术：使得php代码和html代码分离的技术就称为“模板引擎技术”

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p6.png)

**自定义smarty模板技术实现**   

```
<?php

//迷你smarty原理
class MiniSmarty{
    public $var_arr = array();

    public $template_dir = "./view/";
    public $compile_dir = "./view_c/";

    //把外部声明变量设置为当前类内部的成员属性信息var_arr
    function assign($k,$v){
        $this ->var_arr[$k] = $v;
    }

    //显示模板内容
    function display($tpl){
        include $this->compile($tpl);
    }

    //把html模板内容引入，替换{  ---------》< ?php echo
    //                        }  ---------》 ; ? >
    //模板编译，把{}编译为php标签内容
    //$tpl: 被编译模板文件的名称
    function compile($tpl){
        //$tpl = "order.html";
        $tpl_file = $this -> template_dir.$tpl; //模板文件
        $com_file = $this->compile_dir.$tpl.".php"; //编译文件
        //判断编译文件是否存在，如果存在直接调用，并且该编译文件生成后对应的模板文件没有再修改
        //正常情况，编译文件的时间稍大，模板文件时间相对小一些
        if(file_exists($com_file) && (filemtime($tpl_file)<filemtime($com_file))){
            return $com_file;
            exit;
        }
        //把$tpl内容抓取出来，替换内部的{}内容
        $cont = file_get_contents($tpl_file);
        //替换{  ---------》< ?php echo
        //$cont = str_replace("{","< ?php echo ",$cont);
        //在模板中调用的变量即是 当前对象的属性var_arr的元素信息
        //< ?php  echo $this->var_arr['name']; ? >
        $cont = str_replace("{\$","<?php echo \$this->var_arr['",$cont);

        //}  ---------》 ; ? >
        //$cont = str_replace("}","; ? >",$cont);
        $cont = str_replace("}","']; ?>",$cont);

        //把生成好的内容放入一个文件里边，然后引入之
        file_put_contents($com_file, $cont);
        //引入$com_file编译文件
        return $com_file;
    }
}
```

**通过MySmarty对Smarty进行初始化设置**        

```
<?php

//在这个类里边设置smarty公共配置
include "./libs/Smarty.class.php";

class MySmarty extends Smarty{
    function __construct(){

        parent::__construct();  //先执行父类的构造函数，避免被覆盖

        //更改smarty的左右标记
        $this -> left_delimiter = "{";
        $this -> right_delimiter = "}";
        $this -> setTemplateDir('.' . DS . 'view' . DS);//修改模板目录
        $this -> setCompileDir('.' . DS . 'view_c' . DS);//修改编译文件目录
    }
}
```

#### smarty的三种变量使用          
1.$smarty -> assign(名称，值);            
2.超级全局数组变量使用              
$_GET   $_POST   $_SESSION  $_COOKIE  $_ENV   $_SERVER等等       

```

    <body>
        当前用户：{$smarty.session.username}<br />
        名字：{$name}<br />
        颜色：{$color}<br />
        地区：{$smarty.get.addr}<br />
        年龄：{$smarty.get.age}<br />
        {$smarty.now}{*当前时间戳信息*}<br />
        {$smarty.const.HOST}{*获得常量信息*}<br />
        {$smarty.template}{*当前请求的模板*}<br />
        {$smarty.current_dir}{*当前模板路径*}<br />
        {$smarty.version}<br />
        {$smarty.ldelim},{$smarty.rdelim} {*左右标记信息*}<br />

    </body>
```

3.配置变量信息                             
通过配置变量的定义 和 段 的设置可以轻松实现同一页面根据用户不同喜好显示不同的样式。    

**定义**   

```
[shop]
POLICE=京公网安备110000000011号
NETWORK=互联网出版许可证
BROADCAST="广播电视节目制作经营许可证 (京)字第434号"

[car]
POLICE=京公网安备973498378号
NETWORK=互联网出版许可证02
BROADCAST="广播电视节目制作经营许可证 (京)字第978号"
```

通过以下语句引入配置   
{config_load  file="site.conf" section="car"}

```
<body>
        名字：{$name}<br />
        颜色：{$color}<br />
        <h2>显示相关的许可信息</h2>
        {#POLICE#}<br />
        {#NETWORK#}<br />
        {#BROADCAST#}<br />
    </body>
```


#### {}标记冲突的问题       

1.把Smarty的标记变为其他标记    
2.把{}符号变为不同行             
3.利用literal标签把有{}的内容给括起来        


#### 数组/对象变量的使用          

数组：数组[下标]  或 数组.下标          
对象：对象->成员            


```
<?php
include "./MySmarty.class.php";


$smarty = new MySmarty();

//索引数组
$smarty -> assign('fruit',array('banana','pear','apple','grape'));
//关联数组
$smarty -> assign('animal',array('north'=>'tiger','sichuan'=>'panda','helan'=>'pig'));
$smarty -> assign('student',array('first'=>array('tom','jack','mary')));

$smarty -> display('07.html');
```

**访问**        

```
 <body>
        <h2>访问数组元素信息(索引)</h2>
        <div>
            {$fruit[2]}<br />
            {$fruit[3]}<br />
            {$fruit.0}<br />
            {$fruit.1}<br />
        </div>
        <h2>访问数组元素信息(关联)</h2>
        <div>
            {$animal['helan']}<br />
            {$animal.north}<br />
        </div>
        <h2>访问数组元素信息(二维)</h2>
        <div>
            {$student.first.1}<br />
            {$student['first'][2]}<br />
        </div>
    </body>
```

**访问对象** 

```
<?php
include "./MySmarty.class.php";


$smarty = new MySmarty();

class Person{
    public $name = "xiaoming";
    public function run(){
        return "正在跑步。。。";
    }
}

$per = new Person;

$smarty -> assign('per',$per);

$smarty -> display('08.html');

```

在HTML中访问

```
 <body>
        <h2>访问对象信息</h2>

        {$per -> name}<br />
        {$per -> run()}<br />

    </body>
```

#### 遍历数组信息foreach/section   

{foreach  数组  as  $k => $v}         
@iteration  自然数1开始序号           
@index     0开始序号          
@first      如果第一个项目则返回1，否则返回false           
@last      如果最后一个项目则返回1，否则返回false         
@show     判断数组是否为空          
@total     数组的元素个数            

```
 <body>
        <h2>数组遍历</h2>
        {*
            元素值@iteration   获得1开始的自然数序号
            元素值@index       0开始的序号信息
            元素值@first       如果是第一个元素返回1，否则返回0
            元素值@last        如果是最后一个元素返回1，否则返回0
            元素值@total       返回数组元素的总个数
            元素值@show        判断当前数组是否为空
        *}
        <div>
        {foreach $fruit as $k => $v}
            {$v@first}--{$v@index}--{$v@iteration}--{$k}--->{$v}--{$v@last}<br />
        {foreachelse}
            没有任何记录信息<br />
        {/foreach}
        当前数组的总个数：{$v@total}<br />
        当前数组是否为空：{$v@show}
        </div>

        <h2>遍历应用(第一个元素显示特殊背景色)</h2>
        <div>
            {foreach $animal as $kk => $vv}
                {if $vv@first == 1}
                    <p style='background-color:lightblue;'>
                {/if}
                {$kk}---->{$vv}<br />
                {if $vv@first == 1}
                    </p>
                {/if}
            {/foreach}
        </div>
        <h2>遍历应用(隔行显示不同颜色)</h2>
        <div>
            {foreach $animal as $kkk => $vvv}
                {if $vvv@iteration%2 == 0}
                    <p style='background-color:lightgreen;'>
                {/if}
                {$kkk}---->{$vvv}<br />
                {if $vvv@iteration%2 == 0}
                    </p>
                {/if}
            {/foreach}
        </div>
    </body>
```


**二维数组遍历** 

```
 <body>
        <h2>二维数组遍历</h2>
        <div>
            {foreach $student as $k => $v}
                {foreach $v as $kk => $vv}
                    {$kk}--》{$vv}<br />
                {/foreach}
            {/foreach}
        </div>
    </body>
```

**section遍历** 

```
{section  name=”名称”  loop=$animal  start=2  step=2  max=5  show=false}
	{$animal[名称]}
	关键字: 
	{$smarty.section.名称.first}
	{$smarty.section.名称.last}
	{$smarty.section.名称.iteration}
	{$smarty.section.名称.total}
{/section }
```

foreach和section最大的区别是：       
	foreach可以遍历索引和关联数组      
	section只给遍历索引数组             


**for循环**           

```
<body>
        <h2>for循环语句</h2>
        <div>

            {for $i=1 to 10 step=3}
                {$i}<br />
            {/for}

            <hr />
            {*for($m=100 ;  $m<=90;  $m++)*}
            {for $m=100 to 90 step=-2 max=4}
                {$m}<br />
            {/for}

        </div>
    </body>
```

**if分支** 

```
<body>
        <h2>if分支结构语句</h2>
        {*  if  else   elseif *}
        <div>
        {if $age>0 && $age<10}
            儿童<br />
        {elseif $age>=10 && $age<20}
            少年<br />
        {elseif $age>=20 && $age<30}
            青年<br />
        {elseif $age>=30}
            成年<br />
        {/if}
        <hr />
        {if $age+10>=30}
            10年之后就成年了<br />
        {/if}

        </div>
    </body>
```

**复选框、单选按钮、下拉列表**         

```
<?php
include "./MySmarty.class.php";


$smarty = new MySmarty();


$smarty -> assign('hobby_out',array('篮球','足球','排球','棒球'));
$smarty -> assign('hobby_val',array('a','b','c','d'));

$smarty -> assign('val_out',array('A'=>'篮球','B'=>'足球','C'=>'排球','D'=>'棒球'));

$smarty -> assign('sel',array('A','C','D'));

$smarty -> display('13.html');
```


```
<body>
        <h2>复选框设置</h2>
        <div>
            {*<input type="checkbox" name="hobby" value='1'>篮球*}
            {*<input type="checkbox" name="hobby" value='2'>棒球*}

            {html_checkboxes name="hobby" options=$val_out selected=$sel separator="<br />" label_ids=true}

        </div>
    </body>
```


**下拉列表** 

```
<?php
include "./MySmarty.class.php";


$smarty = new MySmarty();


$smarty -> assign('val_out',array('0'=>'请选择','A'=>'小学','B'=>'初中','C'=>'高中','D'=>'大学'));
$smarty -> assign('sel',array('A','D'));

$smarty -> assign('val_out1',array('man','girl','secret'));

$smarty -> display('14.html');
```


```
<body>
        <h2>下拉列表设置</h2>
        <div>

            {html_options name="xueli" options=$val_out multiple="multiple" selected=$sel}

            <hr />
            <select name="sex" >
            <option value="0">请选择</option>
            {html_options options=$val_out1}
            </select>
        </div>
    </body>
```





