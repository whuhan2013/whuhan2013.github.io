---
layout: post
title: PHP语言基础
date: 2016-7-13
categories: blog
tags: [PHP]
description: PHP
---


**Php语言的执行周期**

1.apache来调用php模块         
2.初始化php模块（读取php.ini配置文件，并加装相应的扩展库）      
3.处理php代码        
4.释放php的相应资源             
5.Php模块将处理结果返回到apache             

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p2.png)

Opcode是zend 引擎能够识别并执行的代码。

**PHP是解释性的语言**         

### PHP基础语法

**PHP标记**  

由于php是可以嵌入html代码的服务器端的脚本语言，php可以和html混在一起写，如果没有一个标记以区分哪个是PHP代码，那么php模块如何来处理php代码呢？

标记有四种：          

- <?php  ?> 标准的写法，推荐使用
- <script language='php'> </script>    
- <? ?>  [需要开启short_open_tag] （在php.ini中开启）
- <% %>  [需要开启asp_tags]


**注释:**           
单行注释：#   //             
多行注释： /*  */              


#### 数据类型   

PHP 支持 8 种原始数据类型。

**四种标量类型：**

boolean（布尔型）      
integer（整型）          
float（浮点型，也称作 double)              
string（字符串）          

**两种复合类型：**

array（数组）            
object（对象）       

**最后是两种特殊类型：**

resource（资源）         
NULL（无类型）


如果想查看某个表达式的值和类型，用 var_dump() 函数。
如果只是想得到一个易读懂的类型的表达方式用于调试，用 gettype() 函数。要查看某个类型，不要用 gettype()，而用 is_type 函数


**Integer 整型**                
如果给定的一个数超出了 integer 的范围，将会被解释为 float。同样如果执行的运算结果超出了 integer 范围，也会返回 float。

PHP 中没有整除的运算符。1/2 产生出 float 0.5。值可以舍弃小数部分强制转换为 integer，或者使用 round() 函数可以更好地进行四舍五入。


**String 字符串**           

一个字符串可以用 4 种方式表达：

- 单引号
- 双引号
- heredoc 语法结构
- nowdoc 语法结构（自 PHP 5.3.0 起）

**单引号**

定义一个字符串的最简单的方法是用单引号把它包围起来（字符 '）。

要表达一个单引号自身，需在它的前面加个反斜线（\）来转义。要表达一个反斜线自身，则用两个反斜线（\\）。其它任何方式的反斜线都会被当成反斜线本身：也就是说如果想使用其它转义序列例如 \r 或者 \n，并不代表任何特殊含义，就单纯是这两个字符本身。

Note: 不像双引号和 heredoc 语法结构，在单引号字符串中的变量和特殊字符的转义序列将不会被替换。


**双引号**

如果字符串是包围在双引号（"）中， PHP 将对一些特殊的字符进行解析：

和单引号字符串一样，转义任何其它字符都会导致反斜线被显示出来。PHP 5.1.1 以前，\{$var} 中的反斜线还不会被显示出来。

用双引号定义的字符串最重要的特征是变量会被解析

**Heredoc结构**

第三种表达字符串的方法是用 heredoc 句法结构：<<<。在该运算符之后要提供一个标识符，然后换行。接下来是字符串 string 本身，最后要用前面定义的标识符作为结束标志。

结束时所引用的标识符必须在该行的第一列，而且，标识符的命名也要像其它标签一样遵守 PHP 的规则：只能包含字母、数字和下划线，并且必须以字母和下划线作为开头。

**Warning**
要注意的是结束标识符这行除了可能有一个分号（;）外，绝对不能包含其它字符。这意味着标识符不能缩进，分号的前后也不能有任何空白或制表符。更重要的是结束标识符的前面必须是个被本地操作系统认可的换行，比如在 UNIX 和 Mac OS X 系统中是 \n，而结束定界符（可能其后有个分号）之后也必须紧跟一个换行。
如果不遵守该规则导致结束标识不“干净”，PHP 将认为它不是结束标识符而继续寻找。如果在文件结束前也没有找到一个正确的结束标识符，PHP 将会在最后一行产生一个解析错误。
Heredocs 结构不能用来初始化类的属性。自 PHP 5.3 起，此限制仅对 heredoc 包含变量时有效。

Heredoc 结构就象是没有使用双引号的双引号字符串，这就是说在 heredoc 结构中单引号不用被转义，但是上文中列出的转义序列还可以使用。变量将被替换，但在 heredoc 结构中含有复杂的变量时要格外小心。

Heredoc 结构的字符串示例

```
<?php
$str = <<<EOD
Example of string
spanning multiple lines
using heredoc syntax.
EOD;

/* 含有变量的更复杂示例 */
class foo
{
    var $foo;
    var $bar;

    function foo()
    {
        $this->foo = 'Foo';
        $this->bar = array('Bar1', 'Bar2', 'Bar3');
    }
}

$foo = new foo();
$name = 'MyName';

echo <<<EOT
My name is "$name". I am printing some $foo->foo.
Now, I am printing some {$foo->bar[1]}.
This should print a capital 'A': \x41
EOT;
?>
```


