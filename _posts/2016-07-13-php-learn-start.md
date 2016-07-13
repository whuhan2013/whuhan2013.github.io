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
- '<script language='php'> </script>`    
- <? ?>  [需要开启short_open_tag] （在php.ini中开启）
- <% %>  [需要开启asp_tags]


**注释:**           
单行注释：#   //             
多行注释： /*  */              


### 数据类型   

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


#### Integer 整型               
如果给定的一个数超出了 integer 的范围，将会被解释为 float。同样如果执行的运算结果超出了 integer 范围，也会返回 float。

PHP 中没有整除的运算符。1/2 产生出 float 0.5。值可以舍弃小数部分强制转换为 integer，或者使用 round() 函数可以更好地进行四舍五入。


#### String 字符串       

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


以上例程会输出：

> My name is "MyName". I am printing some Foo.
Now, I am printing some Bar2.
This should print a capital 'A': A

也可以把 Heredoc 结构用在函数参数中来传递数据：

Heredoc 结构在参数中的示例

```
<?php
var_dump(array(<<<EOD
foobar!
EOD
));
?>
```

在 PHP 5.3.0 以后，也可以用 Heredoc 结构来初始化静态变量和类的属性和常量：

使用 Heredoc 结构来初始化静态值

```
<?php
// 静态变量
function foo()
{
    static $bar = <<<LABEL
Nothing in here...
LABEL;
}

// 类的常量、属性
class foo
{
    const BAR = <<<FOOBAR
Constant example
FOOBAR;

    public $baz = <<<FOOBAR
Property example
FOOBAR;
}
?>
```

自 PHP 5.3.0 起还可以在 Heredoc 结构中用双引号来声明标识符：

在 heredoc 结构中使用双引号

```
<?php
echo <<<"FOOBAR"
Hello World!
FOOBAR;
?>
```

**Nowdoc 结构**

就象 heredoc 结构类似于双引号字符串，Nowdoc 结构是类似于单引号字符串的。Nowdoc 结构很象 heredoc 结构，但是 nowdoc 中不进行解析操作。这种结构很适合用于嵌入 PHP 代码或其它大段文本而无需对其中的特殊字符进行转义。与 SGML 的 <![CDATA[ ]]> 结构是用来声明大段的不用解析的文本类似，nowdoc 结构也有相同的特征。

一个 nowdoc 结构也用和 heredocs 结构一样的标记 <<<， 但是跟在后面的标识符要用单引号括起来，即 <<<'EOT'。Heredoc 结构的所有规则也同样适用于 nowdoc 结构，尤其是结束标识符的规则。

Nowdoc 结构字符串示例

```
<?php
$str = <<<'EOD'
Example of string
spanning multiple lines
using nowdoc syntax.
EOD;

/* 含有变量的更复杂的示例 */
class foo
{
    public $foo;
    public $bar;

    function foo()
    {
        $this->foo = 'Foo';
        $this->bar = array('Bar1', 'Bar2', 'Bar3');
    }
}

$foo = new foo();
$name = 'MyName';

echo <<<'EOT'
My name is "$name". I am printing some $foo->foo.
Now, I am printing some {$foo->bar[1]}.
This should not print a capital 'A': \x41
EOT;
?>
```

以上例程会输出：

> My name is "$name". I am printing some $foo->foo.
Now, I am printing some {$foo->bar[1]}.
This should not print a capital 'A': \x41


Note:          
不象 heredoc 结构，nowdoc 结构可以用在任意的静态数据环境中，最典型的示例是用来初始化类的属性或常量：

```
<?php
class foo {
    public $bar = <<<'EOT'
bar
EOT;
}
?>
```

#### 变量解析 

当字符串用双引号或 heredoc 结构定义时，其中的变量将会被解析。

这里共有两种语法规则：一种简单规则，一种复杂规则。简单的语法规则是最常用和最方便的，它可以用最少的代码在一个 string 中嵌入一个变量，一个 array 的值，或一个 object 的属性。

复杂规则语法的显著标记是用花括号包围的表达式。

**简单语法**

当 PHP 解析器遇到一个美元符号（$）时，它会和其它很多解析器一样，去组合尽量多的标识以形成一个合法的变量名。可以用花括号来明确变量名的界线。

```
<?php
$juice = "apple";

echo "He drank some $juice juice.".PHP_EOL;
// Invalid. "s" is a valid character for a variable name, but the variable is $juice.
echo "He drank some juice made of $juices.";
?>
```

以上例程会输出：

> He drank some apple juice.
He drank some juice made of .              

**PHP_EOL**表示换行符，可以在多个平台上适用，增加了代码的可移植性。

类似的，一个 array 索引或一个 object 属性也可被解析。数组索引要用方括号（]）来表示索引结束的边际，对象属性则是和上述的变量规则相同。

**复杂（花括号）语法**

复杂语法不是因为其语法复杂而得名，而是因为它可以使用复杂的表达式。

任何具有 string 表达的标量变量，数组单元或对象属性都可使用此语法。只需简单地像在 string 以外的地方那样写出表达式，然后用花括号 { 和 } 把它括起来即可。由于 { 无法被转义，只有 $ 紧挨着 { 时才会被识别。可以用 {\$ 来表达 {$。下面的示例可以更好的解释：

```
<?php
// 显示所有错误
error_reporting(E_ALL);

$great = 'fantastic';

// 无效，输出: This is { fantastic}
echo "This is { $great}";

// 有效，输出： This is fantastic
echo "This is {$great}";
echo "This is ${great}";

// 有效
echo "This square is {$square->width}00 centimeters broad."; 

// 有效，只有通过花括号语法才能正确解析带引号的键名
echo "This works: {$arr['key']}";

// 有效
echo "This works: {$arr[4][3]}";

// 这是错误的表达式，因为就象 $foo[bar] 的格式在字符串以外也是错的一样。
// 换句话说，只有在 PHP 能找到常量 foo 的前提下才会正常工作；这里会产生一个
// E_NOTICE (undefined constant) 级别的错误。
echo "This is wrong: {$arr[foo][3]}"; 

// 有效，当在字符串中使用多重数组时，一定要用括号将它括起来
echo "This works: {$arr['foo'][3]}";

// 有效
echo "This works: " . $arr['foo'][3];

echo "This works too: {$obj->values[3]->name}";

echo "This is the value of the var named $name: {${$name}}";

echo "This is the value of the var named by the return value of getName(): {${getName()}}";

echo "This is the value of the var named by the return value of \$object->getName(): {${$object->getName()}}";

// 无效，输出： This is the return value of getName(): {getName()}
echo "This is the return value of getName(): {getName()}";
?>
```

**Note:**                 
函数、方法、静态类变量和类常量只有在 PHP 5 以后才可在 {$} 中使用。然而，只有在该字符串被定义的命名空间中才可以将其值作为变量名来访问。只单一使用花括号 ({}) 无法处理从函数或方法的返回值或者类常量以及类静态变量的值。          


**存取和修改字符串中的字符 **

string 中的字符可以通过一个从 0 开始的下标，用类似 array 结构中的方括号包含对应的数字来访问和修改，比如 $str[42]。可以把 string 当成字符组成的 array。函数 substr() 和 substr_replace() 可用于操作多于一个字符的情况。

**Note:** string 也可用花括号访问，比如 $str{42}。

**Warning**           
用超出字符串长度的下标写入将会拉长该字符串并以空格填充。非整数类型下标会被转换成整数。非法下标类型会产生一个 E_NOTICE 级别错误。用负数下标写入字符串时会产生一个 E_NOTICE 级别错误，用负数下标读取字符串时返回空字符串。写入时只用到了赋值字符串的第一个字符。用空字符串赋值则赋给的值是 NULL 字符。

**Warning**          
PHP 的字符串在内部是字节组成的数组。因此用花括号访问或修改字符串对多字节字符集很不安全。仅应对单字节编码例如 ISO-8859-1 的字符串进行此类操作

一些字符串示例

```
<?php
// 取得字符串的第一个字符
$str = 'This is a test.';
$first = $str[0];

// 取得字符串的第三个字符
$third = $str[2];

// 取得字符串的最后一个字符
$str = 'This is still a test.';
$last = $str[strlen($str)-1]; 

// 修改字符串的最后一个字符
$str = 'Look at the sea';
$str[strlen($str)-1] = 'e';

?>
```

自 PHP 5.4 起字符串下标必须为整数或可转换为整数的字符串，否则会发出警告。之前例如 "foo" 的下标会无声地转换成 0。

**有用的函数和运算符**

字符串可以用 '.'（点）运算符连接起来，注意 '+'（加号）运算符没有这个功能。更多信息参考字符串运算符。

对于 string 的操作有很多有用的函数。

可以参考字符串函数了解大部分函数，高级的查找与替换功能可以参考正则表达式函数或 Perl 兼容正则表达式函数。

另外还有 URL 字符串函数，也有加密／解密字符串的函数（mcrypt 和 mhash）。

最后，可以参考字符类型函数。

**转换成字符串**

一个值可以通过在其前面加上 (string) 或用 strval() 函数来转变成字符串。在一个需要字符串的表达式中，会自动转换为 string。比如在使用函数 echo 或 print 时，或在一个变量和一个 string 进行比较时，就会发生这种转换。类型和类型转换可以更好的解释下面的事情，也可参考函数 settype()。

一个布尔值 boolean 的 TRUE 被转换成 string 的 "1"。Boolean 的 FALSE 被转换成 ""（空字符串）。这种转换可以在 boolean 和 string 之间相互进行。

一个整数 integer 或浮点数 float 被转换为数字的字面样式的 string（包括 float 中的指数部分）。使用指数计数法的浮点数（4.1E+6）也可转换。

数组 array 总是转换成字符串 "Array"，因此，echo 和 print 无法显示出该数组的内容。要显示某个单元，可以用 echo $arr['foo'] 这种结构。要显示整个数组内容见下文。

在 PHP 4 中对象 object 总是被转换成字符串 "Object"，如果为了调试原因需要打印出对象的值，请继续阅读下文。为了得到对象的类的名称，可以用 get_class() 函数。自 PHP 5 起，适当时可以用 __toString 方法。

资源 resource 总会被转变成 "Resource id #1" 这种结构的字符串，其中的 1 是 PHP 在运行时分配给该 resource 的唯一值。不要依赖此结构，可能会有变更。要得到一个 resource 的类型，可以用函数 get_resource_type()。

NULL 总是被转变成空字符串。

如上面所说的，直接把 array，object 或 resource 转换成 string 不会得到除了其类型之外的任何有用信息。可以使用函数 print_r() 和 var_dump() 列出这些类型的内容。

大部分的 PHP 值可以转变成 string 来永久保存，这被称作串行化，可以用函数 serialize() 来实现。如果 PHP 引擎设定支持 WDDX，PHP 值也可被串行化为格式良好的 XML 文本。


**字符串转换为数值**

当一个字符串被当作一个数值来取值，其结果和类型如下：

如果该字符串没有包含 '.'，'e' 或 'E' 并且其数字值在整型的范围之内（由 PHP_INT_MAX 所定义），该字符串将被当成 integer 来取值。其它所有情况下都被作为 float 来取值。

该字符串的开始部分决定了它的值。如果该字符串以合法的数值开始，则使用该数值。否则其值为 0（零）。合法数值由可选的正负号，后面跟着一个或多个数字（可能有小数点），再跟着可选的指数部分。指数部分由 'e' 或 'E' 后面跟着一个或多个数字构成。

```
<?php
$foo = 1 + "10.5";                // $foo is float (11.5)
$foo = 1 + "-1.3e3";              // $foo is float (-1299)
$foo = 1 + "bob-1.3e3";           // $foo is integer (1)
$foo = 1 + "bob3";                // $foo is integer (1)
$foo = 1 + "10 Small Pigs";       // $foo is integer (11)
$foo = 4 + "10.2 Little Piggies"; // $foo is float (14.2)
$foo = "10.0 pigs " + 1;          // $foo is float (11)
$foo = "10.0 pigs " + 1.0;        // $foo is float (11)     
?>
```

本节中的示例可以通过复制／粘贴到下面的代码中来显示：

```
<?php
echo "\$foo==$foo; type is " . gettype ($foo) . "<br />\n";
?>
```

不要想像在 C 语言中的那样，通过将一个字符转换成整数以得到其代码。使用函数 ord() 和 chr() 实现 ASCII 码和字符间的转换。


#### Array 数组 

定义数组 array() ¶

可以用 array() 语言结构来新建一个数组。它接受任意数量用逗号分隔的 键（key） => 值（value）对。

```
<?php
$array = array(
    "foo" => "bar",
    "bar" => "foo",
);

// 自 PHP 5.4 起
$array = [
    "foo" => "bar",
    "bar" => "foo",
];
?>
```

key 可以是 integer 或者 string。value 可以是任意类型。

此外 key 会有如下的强制转换：

- 包含有合法整型值的字符串会被转换为整型。例如键名 "8" 实际会被储存为 8。但是 "08" 则不会强制转换，因为其不是一个合法的十进制数值。
- 浮点数也会被转换为整型，意味着其小数部分会被舍去。例如键名 8.7 实际会被储存为 8。
- 布尔值也会被转换成整型。即键名 true 实际会被储存为 1 而键名 false 会被储存为 0。
- Null 会被转换为空字符串，即键名 null 实际会被储存为 ""。
- 数组和对象不能被用为键名。坚持这么做会导致警告：Illegal offset type。
- 如果在数组定义中多个单元都使用了同一个键名，则只使用了最后一个，之前的都被覆盖了。

没有键名的索引数组

```
<?php
$array = array("foo", "bar", "hallo", "world");
var_dump($array);
?>
```

以上例程会输出：

```
array(4) {
  [0]=>
  string(3) "foo"
  [1]=>
  string(3) "bar"
  [2]=>
  string(5) "hallo"
  [3]=>
  string(5) "world"
}
```

**访问**         
方括号和花括号可以互换使用来访问数组单元（例如 $array[42] 和 $array{42} 在上例中效果相同）。

自 PHP 5.4 起可以用数组间接引用函数或方法调用的结果。之前只能通过一个临时变量。

自 PHP 5.5 起可以用数组间接引用一个数组原型。

**数组间接引用**

```
<?php
function getArray() {
    return array(1, 2, 3);
}

// on PHP 5.4
$secondElement = getArray()[1];

// previously
$tmp = getArray();
$secondElement = $tmp[1];

// or
list(, $secondElement) = getArray();
?>
```

**Note:**       
试图访问一个未定义的数组键名与访问任何未定义变量一样：会导致 E_NOTICE 级别错误信息，其结果为 NULL。


**用方括号的语法新建／修改**           
要修改某个值，通过其键名给该单元赋一个新值。要删除某键值对，对其调用 unset() 函数。

```
<?php
$arr = array(5 => 1, 12 => 2);

$arr[] = 56;    // This is the same as $arr[13] = 56;
                // at this point of the script

$arr["x"] = 42; // This adds a new element to
                // the array with key "x"
                
unset($arr[5]); // This removes the element from the array

unset($arr);    // This deletes the whole array
?>
```

**Note:**    
如上所述，如果给出方括号但没有指定键名，则取当前最大整数索引值，新的键名将是该值加上 1（但是最小为 0）。如果当前还没有整数索引，则键名将为 0。             
注意这里所使用的最大整数键名不一定当前就在数组中。它只要在上次数组重新生成索引后曾经存在过就行了。以下面的例子来说明：                         

```
<?php
// 创建一个简单的数组
$array = array(1, 2, 3, 4, 5);
print_r($array);

// 现在删除其中的所有元素，但保持数组本身不变:
foreach ($array as $i => $value) {
    unset($array[$i]);
}
print_r($array);

// 添加一个单元（注意新的键名是 5，而不是你可能以为的 0）
$array[] = 6;
print_r($array);

// 重新索引：
$array = array_values($array);
$array[] = 7;
print_r($array);
?>
```

以上例程会输出：

```
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
    [3] => 4
    [4] => 5
)
Array
(
)
Array
(
    [5] => 6
)
Array
(
    [0] => 6
    [1] => 7
)
```

**实用函数**               

unset() 函数允许删除数组中的某个键。但要注意数组将不会重建索引。如果需要删除后重建索引，可以用 array_values() 函数。

foreach 控制结构是专门用于数组的。它提供了一个简单的方法来遍历数组。


**Note:** 重申一次，在双引号字符串中，不给索引加上引号是合法的因此 "$foo[bar]" 是合法的（“合法”的原文为 valid。在实际测试中，这么做确实可以访问数组的该元素，但是会报一个常量未定义的 notice。无论如何，强烈建议不要使用 $foo[bar]这样的写法，而要使用 $foo['bar'] 来访问数组中元素。--haohappy 注）。至于为什么参见以上的例子和字符串中的变量解析中的解释。

**转换为数组**

对于任意 integer，float，string，boolean 和 resource 类型，如果将一个值转换为数组，将得到一个仅有一个元素的数组，其下标为 0，该元素即为此标量的值。换句话说，(array)$scalarValue 与 array($scalarValue) 完全一样。

如果一个 object 类型转换为 array，则结果为一个数组，其单元为该对象的属性。键名将为成员变量名，不过有几点例外：整数属性不可访问；私有变量前会加上类名作前缀；保护变量前会加上一个 '*' 做前缀。这些前缀的前后都各有一个 NULL 字符。这会导致一些不可预知的行为：

```
<?php

class A {
    private $A; // This will become '\0A\0A'
}

class B extends A {
    private $A; // This will become '\0B\0A'
    public $AA; // This will become 'AA'
}

var_dump((array) new B());
?>
```

**比较**

可以用 array_diff() 和数组运算符来比较数组。


#### Object 对象 

**对象初始化 **

要创建一个新的对象 object，使用 new 语句实例化一个类：

```
<?php
class foo
{
    function do_foo()
    {
        echo "Doing foo."; 
    }
}

$bar = new foo;
$bar->do_foo();
?>
```

详细讨论参见手册中类与对象章节。

**转换为对象 **

如果将一个对象转换成对象，它将不会有任何变化。如果其它任何类型的值被转换成对象，将会创建一个内置类 stdClass 的实例。如果该值为 NULL，则新的实例为空。数组转换成对象将使键名成为属性名并具有相对应的值。对于任何其它的值，名为 scalar 的成员变量将包含该值。

```
<?php
$obj = (object) 'ciao';
echo $obj->scalar;  // outputs 'ciao'
?>
```

#### Resource 资源类型 

资源 resource 是一种特殊变量，保存了到外部资源的一个引用。资源是通过专门的函数来建立和使用的。所有这些函数及其相应资源类型见附录。

参见 get_resource_type()。

**转换为资源**

由于资源类型变量保存有为打开文件、数据库连接、图形画布区域等的特殊句柄，因此将其它类型的值转换为资源没有意义。

**释放资源**

由于 PHP 4 Zend 引擎引进了引用计数系统，可以自动检测到一个资源不再被引用了（和 Java 一样）。这种情况下此资源使用的所有外部资源都会被垃圾回收系统释放。因此，很少需要手工释放内存。

Note: 持久数据库连接比较特殊，它们不会被垃圾回收系统销毁。参见数据库永久连接一章。



#### NULL

特殊的 NULL 值表示一个变量没有值。NULL 类型唯一可能的值就是 NULL。

在下列情况下一个变量被认为是 NULL：

- 被赋值为 NULL。

- 尚未被赋值。

- 被 unset()。

**语法**

NULL 类型只有一个值，就是不区分大小写的常量 NULL。

```
<?php
$var = NULL;       
?>
```

参见 is_null() 和 unset()。

**转换到 NULL** 

使用 (unset) $var 将一个变量转换为 null 将不会删除该变量或 unset 其值。仅是返回 NULL 值而已。

#### Callback 回调类型 

自 PHP 5.4 起可用 callable 类型指定回调类型 callback。本文档基于同样理由使用 callback 类型信息。

一些函数如 call_user_func() 或 usort() 可以接受用户自定义的回调函数作为参数。回调函数不止可以是简单函数，还可以是对象的方法，包括静态类方法。

**传递** 

一个 PHP 的函数以 string 类型传递其名称。可以使用任何内置或用户自定义函数，但除了语言结构例如：array()，echo，empty()，eval()，exit()，isset()，list()，print 或 unset()。

一个已实例化的对象的方法被作为数组传递，下标 0 包含该对象，下标 1 包含方法名。

静态类方法也可不经实例化该类的对象而传递，只要在下标 0 中包含类名而不是对象。自 PHP 5.2.3 起，也可以传递 'ClassName::methodName'。

除了普通的用户自定义函数外，create_function() 可以用来创建一个匿名回调函数。自 PHP 5.3.0 起也可传递 closure 给回调参数。

```
<?php 

// An example callback function
function my_callback_function() {
    echo 'hello world!';
}

// An example callback method
class MyClass {
    static function myCallbackMethod() {
        echo 'Hello World!';
    }
}

// Type 1: Simple callback
call_user_func('my_callback_function'); 

// Type 2: Static class method call
call_user_func(array('MyClass', 'myCallbackMethod')); 

// Type 3: Object method call
$obj = new MyClass();
call_user_func(array($obj, 'myCallbackMethod'));

// Type 4: Static class method call (As of PHP 5.2.3)
call_user_func('MyClass::myCallbackMethod');

// Type 5: Relative static class method call (As of PHP 5.3.0)
class A {
    public static function who() {
        echo "A\n";
    }
}

class B extends A {
    public static function who() {
        echo "B\n";
    }
}

call_user_func(array('B', 'parent::who')); // A
?>
```

#### 伪类型    
有时候，我们需要在程序或手册中描述数据的类型，这就是伪类型。     
Number 数值型，如max函数         
Mixed 混合类型（不确定），如var_dump函数          
Callback 回调函数，如array_map              
Void 空，如echo和pi                  


#### 类型转换的判别    

PHP 在变量定义中不需要（或不支持）明确的类型定义；变量类型是根据使用该变量的上下文所决定的。也就是说，如果把一个 string 值赋给变量 $var，$var 就成了一个 string。如果又把一个integer 赋给 $var，那它就成了一个integer。

PHP 的自动类型转换的一个例子是加法运算符“+”。如果任何一个操作数是float，则所有的操作数都被当成float，结果也是float。否则操作数会被解释为integer，结果也是integer。注意这并没有改变这些操作数本身的类型；改变的仅是这些操作数如何被求值以及表达式本身的类型。


```
<?php
$foo = "0";  // $foo 是字符串 (ASCII 48)
$foo += 2;   // $foo 现在是一个整数 (2)
$foo = $foo + 1.3;  // $foo 现在是一个浮点数 (3.3)
$foo = 5 + "10 Little Piggies"; // $foo 是整数 (15)
$foo = 5 + "10 Small Pigs";     // $foo 是整数 (15)
?>
```

如果上面两个例子看上去古怪的话，参见字符串转换为数值。

如果要强制将一个变量当作某种类型来求值，参见类型强制转换一节。如果要改变一个变量的类型，参见 settype()。

如果想要测试本节中任何例子的话，可以用 var_dump() 函数

**Note:**        
自动转换为 数组 的行为目前没有定义。      
此外，由于 PHP 支持使用和数组下标同样的语法访问字符串下标，以下例子在所有 PHP 版本中都有效：

```
<?php
$a    = 'car'; // $a is a string
$a[0] = 'b';   // $a is still a string
echo $a;       // bar
?>
```

**类型强制转换**

PHP 中的类型强制转换和 C 中的非常像：在要转换的变量之前加上用括号括起来的目标类型。

```
<?php
$foo = 10;   // $foo is an integer
$bar = (boolean) $foo;   // $bar is a boolean
?>
```

允许的强制转换有：

- (int), (integer) - 转换为整形 integer
- (bool), (boolean) - 转换为布尔类型 boolean
- (float), (double), (real) - 转换为浮点型 float
- (string) - 转换为字符串 string
- (array) - 转换为数组 array
- (object) - 转换为对象 object
- (unset) - 转换为 NULL (PHP 5)
- (binary) 转换和 b 前缀转换支持为 PHP 5.2.1 新增。

注意在括号内允许有空格和制表符，所以下面两个例子功能相同：

```
<?php
$foo = (int) $bar;
$foo = ( int ) $bar;
?>
```

将字符串文字和变量转换为二进制字符串：

```
<?php
$binary = (binary)$string;
$binary = b"binary string";
?>
```

Note:            
可以将变量放置在双引号中的方式来代替将变量转换成字符串：     

```
<?php
$foo = 10;            // $foo 是一个整数
$str = "$foo";        // $str 是一个字符串
$fst = (string) $foo; // $fst 也是一个字符串

// 输出 "they are the same"
if ($fst === $str) {
    echo "they are the same";
}
?>
```