---
layout: post
title: PHP语言基础(三)
date: 2016-7-14
categories: blog
tags: [PHP]
description: PHP
---

### 函数            

**关于函数的函数**        

- function_exists(); 判断一个函数是否被定义了 
- create_function()，创建一个函数，通过内置的函数的形式，自动完成函数的创建!
函数名 = create_function(‘参数列表’, ‘函数体内容’);

**大家可以采用三种方法得到函数：**          
1，function创建一个普通函数            
2，function声明一个匿名函数              
3, 使用create_function来创建函数！有函数名                    


**魔术常量,__FUNCTION__**            
在函数，获得当前的函数名的魔术常量！

#### 匿名函数      
没有名字的函数               
是 Closure类的对象，实现匿名函数的基础！
在管理一个对象（数据）。        

```
<?php
$greet = function($name)
{
    printf("Hello %s\r\n", $name);
};

$greet('World');
$greet('PHP');
?>
```

**USE语法**      
use，可以使得匿名函数内部，来使用匿名函数外部作用域变量的数据


use 使用的变量，也分为 引用传递和值传递！    
默认是值传递：  注意，不是找全局，而是找外部！


### 数组遍历    

- current()，获得当前元素的值
- key()：获得当前元素的键。如果指针已经非法，返回NULL。用来判断是否存在元素了
- next()函数，可以完成指针的移动！

```
foreach(遍历的数组 as 键变量 => 值变量 )
{
    循环体 
}
```

**使用foreach的注意事项**                        
- 修改$value 是不会影响到原数组的值的！             
- 保存值的变量，支持引用传递,在$value 前增加  &,修改$value则会改变结果,键变量不能引用传递     
- foreach遍历的是原数组的拷贝，而不是在原数组上做的操作               
在遍历的过程中，如果对原数组做操作，是不会影响到遍历结果的         
- foreach也是一个循环结构：             
break，continue，替代语法都存在


**数组指针的操作**                
利用php的内置函数：

- key，
- current
- next();
- prev();移动到上一个
- reset();//重置，移动到第一个元素
- end();//移动到最后一个元素上                     

注意一旦指针位置非法，则不能做相对移动（next,prev）,可以绝对移动（reset，end）

reset,使用频率较高！                           
each()，集合了 key，current，和next三者的功能！将当前元素信息获得后，移动指针到下一个元素上！
元素信息数组 = each($arr).移动指针                         

**注意**，元素信息数组，是两种表示方案：索引和关联：其中：          
索引：0,1分别 表示 键和值                 
关联：key，value分别表示 键和值                     

**each+while+list遍历数组**             

```
先执行each，将结果赋值给$element，再判断是否成立
while($element = each($names)) {
  //得到键变量和值变量的工作
  $key = $element['key'];//$element[0]
  $value = $element['value'];//$element[1]

  //循环体操作key和value
  var_dump($key, $value);
  echo '<br>';

}
```

**利用 list结构**

each的返回值就包含了索引数组         
0为键，1为值！       
利用list简化的结果：                

```
while(list($key, $value) = each($names)) {
    //循环体操作key和value
    var_dump($key, $value);
    echo '<br>';

}
```


#### 数组函数      

- range()函数，可以得到某个范围内的元素数组
- array_merge(); 数组合并，合并多个             
下标重复会怎么样？        
数值索引：完全重新索引！                
字符下标：后出现的元素值会覆盖前面的元素值！        

- array_rand(数组，个数);随机地从数组内取得元素，取得是下标！                         
如果多个，返回随机下标的集合！结果是被排序之后的，从小到大！           

- shuffle(&$arr).打乱数组内元素的顺序               
注意，参数为引用传递！会打乱原数组  


**键值操作**        

- array_keys(); 取得所有的键
- array_values();取得所有的值
- in_array();是否存在某个值
- array_key_exists();某个键是否存在      
判断某个元素是否存在，典型的是使用isset()来判断                        

- array_combine();利用两个数组合并成一个数组，其中一个作为键，另一个作为值！        
- rray_fill();填充数组            
数组 = array_fill(起始下标，填充的元素个数，填充的值);


**拆分合并**     

- array_merge()
- array_chunk();拆分数组，原则是子数组内的元素个数！         
- explode()，将字符串依据某个分隔符，分割成多个数组
- implode()，将数组内的元素，利用某个分隔符，连接成一个字符串！  


- array_intersect($arr1, $arr2)                    计算两个数组的交集，找到在$arr1中存在，并也在$arr2中存在的元素，数据是出现在第一个参数中的： 
- array_diff($arr1, $arr2)             
计算两个数组的差集。找到在arr1中存在，但是在arr2中不存在的元素！            


**模拟栈和队列**          

- 入栈：array_push，在桟尾放入元素！
- 出栈：array_pop           

数组是引用传递

与删除元素与增加元素使用[]不同，array_push与array_pop会重新索引，保证所有的元素都是由0开始的逐一递增！

- 入队列：array_push()在数组的尾端将数据压入数组
- 出队列：array_shift();在数组的顶端，将数据取出


**可以使用回调函数**   

- array_map        
可以针对数组的每一个元素的值，去调用某个函数！比较适合索引数组的处理！
**典型功能：**
一次性处理多个数组，可以同时将多个数组的同位置的参数，传递到一个函数内！       
处理多个数组时，默认操作是将三个数组合并成一个数组！形参一个二维数组！            

- array_walk            
类似于foreach           

```
array_walk($yuwen, function(& $value, $key) {
    $value += 10;
    var_dump($value, $key);
    echo '<br>';
});
```

#### 排序类      


- sort()，按照值，升序，不保持键值关联        
排序函数都是引用传值！         

- ksort()按照键，升序
- rsort()按照值，降序，不保持关联 
- krsort();按照键，降序                
- asort按照值，升序，保持关联            
- arsort按照值，降序，保持关联            
- natsort()自然数排序        
可以利用计算出来的自然数，对数据进行排序！        
- usort()自定义排序             
u user 用户自定义，用户自定义的元素之间的大小关系！          
用户提供一个比较两个元素大小的函数了，并可以告知php元素的大小关系！







