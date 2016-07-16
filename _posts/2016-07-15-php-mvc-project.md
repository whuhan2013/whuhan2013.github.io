---
layout: post
title: PHP之MVC项目实战
date: 2016-7-15
categories: blog
tags: [PHP]
description: PHP
---

**本文主要包括以下内容**       

1. 类文件自动加载   
2. 路径管理       
3. 页面跳转     
4. 注册自动加载方法           
5. 配置文件系统        
6. cookie         
7. session

#### 类文件自动加载      

在PHP中使用别的类时，需要载入类文件，如果类很多的话，需要重复写很多代码，所以利用__autoload魔法方法实现自动加载 

```
/**
 * 自动加载函数
 *
 * @param $class_name string 需要的类名
 */
function __autoload($class_name) {
//	echo $class_name, '&nbsp;';
	//特例
	$map = array(
		'MySQLDB' => FRAME_DIR . 'MySQLDB.class.php',
		'Model' => FRAME_DIR . 'Model.class.php'
	);//该数组，将所有的有限的特例，类与类名的映射，完成一个列表
	//判断当前所需要加载的类是否是特例类
	if( isset($map[$class_name])) {
		//存在该元素，是特例
		//直接载入
		require $map[$class_name];
	}
	//规律
	elseif (substr($class_name, -10) == 'Controller') {
		//控制器
		require CURR_CONT_DIR . $class_name . '.class.php';
	} elseif (substr($class_name, -5) == 'Model') {
		//模型
		require MODEL_DIR . $class_name . '.class.php';
	}
}
```

#### 路径管理

项目中，使用常量的形式管理路径！ 
使用绝对路径！      
尽量自动获得！           

如果目录之前进行拼凑，一定会使用到目录分隔符！（PATH_SEPARATOR，路径分隔符）       ，不同的操作系统对目录分隔符的支持是不同的！           
windows 支持 \（反斜杠） 和 /（斜杠），默认是反斜杠\。           
linux 支持 / （斜杠）           
因此程序中多见 /斜杠！         
除此，还有一个更好的方法：               
利用 预定义 常量：DIRECTORY_SEPARATOR，目录分隔符！               

```
//管理路径常量
define('DS', DIRECTORY_SEPARATOR);//简化目录分隔符名称长度！
define('ROOT_DIR', dirname(__FILE__) . DS);//根
define('APP_DIR', ROOT_DIR . 'app' . DS);//应用程序
define('CONT_DIR', APP_DIR . 'controller' . DS);//控制器
define('CURR_CONT_DIR', CONT_DIR . PLATFORM . DS);//当前控制器
define('VIEW_DIR', APP_DIR . 'view' . DS);//视图
define('CURR_VIEW_DIR', VIEW_DIR . PLATFORM . DS);//当前视图
define('MODEL_DIR', APP_DIR . 'model' . DS);//模型路径
define('FRAME_DIR', ROOT_DIR . 'framework' . DS);//框架路径
```

#### 页面跳转       

**header(‘Location: url’);**                 
优势：在于立即跳转！                 
劣势：没有办法在跳转前给出提示！         

header功能是，发送响应头信息！相应头信息，是相应信息的一部分！通知浏览器应该做哪些事情的部分！但是要求，相应头信息，要先于响应主体（相应信息的其他部分）先被发送到浏览器！              
因此，无论如何也是看不到echo的提示：

因此，编程上建议在使用header函数时，前面不应该有任何的输出！包含html输出和phpecho输出！

**location.href=’url’,javascript**                 
提示是，样式不易控！（可以用弹出层）

但是，js的支持，需要浏览器支持才可以!

**meta：Refresh**             
refresh 是刷新的意思，可以提供一个秒数，刷新的间隔！              
有需要当前页面执行结束后才会刷新，因此容易给出提示，包括提示的样式！       

因此项目中典型的提示跳转都由  refresh完成！           

**格外注意：**
跳转的代码执行结束后，脚本是没有停止的！       
因此，跳转的代码后边强制脚本停止！     


#### 注册自动加载方法      

利用 系统函数：          
spl_autoload_registser（）     

如果需要注册的是一个函数：直接提供函数名即可             
如果是方法的话：需要给出类或者对象（是否是静态） 和 方法名！        
此时需要一个数组，使用两个元素，分别表示     
array(类名或对象，方法名);

```
spl_autoload_register(array('Framework', 'itcast_autoload'));
```


#### 配置文件系统         
一，增加一个文件，保存配置信息         
二，项目运行时，将配置文件载入，就可以使用配置信息           


**增加配置文件**         
在app目录增加一个config子目录，用于管理配置文件：      

```
<?php
//配置文件

return array(
	'database' => array(
		'host'=>'127.0.0.1',
		'port'=>'3306',
		'user'=>'root',
		'pass'=>'root',
		'charset'=>'utf8',
		'dbname'=>'itcast_shop',
	),//数据库组
	'app' => array(),//应用程序项目组
	'back' => array(),//后台
	'front' => array(),
);
```

**载入配置文件**                             
增加一个框架级别的基础操作，载入配置信息！                 
在framework/Framework.class.php                   
增一个方法，在run方法中执行即可！             

考虑获得数据的变量：
保证保存配置信息的变量的全局性！使用$GLOBALS

```
/**
	 * 载入路径常量
	 */
	private static function loadConfig() {
		$GLOBALS['config'] = require CONFIG_DIR . 'app.config.php';
	}
```

**使用配置信息**       
在基础模型实例化MysQLDB类时需要使用       
framework/Model.class.php       

```
protected function initLink() {
		//require './framework/MySQLDB.class.php';
		// $options = array(
		// 	'host'=>'127.0.0.1',
		// 	'port'=>'3306',
		// 	'user'=>'root',
		// 	'pass'=>'root',
		// 	'charset'=>'utf8',
		// 	'dbname'=>'itcast_shop'
		// );
		$this->db = MySQLDB::getInstance($GLOBALS['config']['database']);
	}
```



#### Cookie       
浏览器的技术，可以在浏览器上保存数据的一门技术！cookie就是指的是浏览器上保存的数据！
PHP支持cookie技术！php可以向浏览器发出指令，从而将数据保存到浏览器上！

浏览器负责保存数据，而php负责控制浏览器保存那些数据！
（php在是使用浏览器上cookie技术）


保存在浏览器上的cookie数据，可以在浏览器每次项服务器请求时，都可以携带该数据，向服务器发出请求，此时服务器上的脚本就可以获得该数据！

**基本使用**                                          
设置cookie变量，增，改，删             
利用内部函数 setcookie完成                  
setcookie(名字，值)             

取得cookie变量，读       
使用预定义数组变量：$_COOKIE         
该变量内保存所有从浏览器请求时所携带的cookie数据！       
每个元素就是一个cookie变量数据！下标是名字，值，就是值！             

**高级使用**                  
1，cookie数据只能是字符串数据！            
2. setcookie函数，可以完成增，修改，删除！         
不存在，则增加，存在则修改！               
删除，可以采用将值，置空的形式！                 
3. cookie变量的失效期      
cookie数据存在有效期的概念：          
默认，临时cookie。会保存到浏览器关闭！                        
同时，支持，增加setcookie的第三个参数，来修改cookie变量的有效期。有效期的表示方式，是一个时间戳，表示到哪个时间点，失效！         
php可以通过 time()函数，获得当前的时间戳，time()增加增量的形式延长cookie时间！          

```
setcookie('name','test',time()+60*60);
```
4，cookie存在有效路径的概念      
cookie变量是只在当前目录，及其后代目录才会生效！      
 
test/下设置           
test/sub/下可以访问            

可以更改cookie数据的有效路径：         
通过setcookie的第四个参数做修改：           

/表示站点根目录有效！整站有效！ 

5,cookie子域名的概念        

cookie是严格区分域名的。

支持在子域名之间是可以共享的：
利用第五个参数设置

有效期，有效路径，有效子域名！              

6，$_COOKIE是捕获不了当前脚本所设置的cookie变量的！             
$_COOKIE是，浏览器请求时所携带的所有cookie！      
当前设置的在下次使用请求才好用！         

**问题**      
由于COOKIE似乎逐渐退出历史舞台，不再的浏览器对于cookie的支持也不一致，所以以后慎用吧。



#### session                 
场景：
cookie的问题
由于是数据本身是在浏览器端：
数据的安全性问题！
数据总要在请求时携带！

怎么解决，注意保持在浏览器的多次请求间共享数据！

将数据放在服务器端，同时是数据区分浏览器，在浏览器的多次请求间共享数据！               

在服务器上，为来访的每台浏览器增加一个数据空间，然后为这些数据空间分配不同唯一的标识！为每个浏览器分配一个唯一的标识，该标志应该服务器端数据库空间的标识应该一一对应        
要求，浏览器每次请求时携带标识,此时服务器可以获得标识，利用标识确定数据空间，但却请求的所有的数据处理，都在当前的确定的空间内完成！          

将服务器分配给浏览器的唯一标识存在浏览器的cookie内，可以保证浏览器每次来时都携带！           
服务器为每一个新浏览器访问（没有确定标识的浏览器），确定 标识，和在服务器上生成一个唯一的数据空间！     


**基本使用**             
直接操作$_SESSION数组，就可以完成session数据的存，取！
每个session数据，就对应$_SESSION内的一个元素！对元素操作，就是对session数据做操作！

但是，session技术，包括生产session标识，开辟session数据空间，为浏览器分配session标识等等，都需要PHP的session机制支持！            
因此，需要先开启session的支持，才能操作$_SESSION变量，从而去操作session数据！           

开启：         
session_start();         

操作：      
$_SESSION;               

```
session_start();
		if(isset($_SESSION['is_login']) && $_SESSION['is_login'] =='yes') {
			//继续执行
		} else {
			die('没有登陆');
		}
```

**基本原理**

浏览器端cookie中保存的sessionID：

当前浏览器第一次对服务器发出请求时，服务器不能确定浏览器的标识 
会重新生成一个唯一标识，以cookie的形式保存到浏览器端！            
其中默认的cookie变量名为：PHPSESSID。             
该cookie标量，也被称之为 sessionID！             

当浏览器拥有了sessionid这个cookie变量后，接下来的请求都会携带该ID发出请求：  

服务器的端的是session数据空间           
默认情况下，php，会将保存session数据的空间，生成一个文件来完成！通过文件的名称来区分属于哪个ID的！
默认的被保存在服务器操作系统的临时目录内：          


**详细介绍**       

**自动开启**   
session可以自动开启！在当前的脚本执行之前，就完成开启！     
通过php的配置文件，修改即可！                
修改为session.auto_start=1                    

注意，在session已经开启的情况下再开启，则会触发错误！       
因此常用@session_start();开启;


**session_start()前的输出问题**         
session_start前也不应该有任何的输出，因为可能会使用到响应头信息（类似于header函数）         

**$_SESSION的使用**            
$_SESSION 变量只能是字符串下标数组，不支持数值下标数组！            

没有session_start时是没有$_SESSION的。                                  
但是语法上可以向一个普通数组一样处理，此时不会对_SESSION内的数据做保存！            

**删除session**            
删除某个session数据               
unset($_SESSION[‘key’]);          
 
删除所有的session数据              
$_SESSION=array();        

注意，不能unset($_SESSION);         

**删除session的文件**          
session_destroy()                    
session_destroy()只删除文件，不处理$_SESSION变量！           
因此$_SESSION是有值的！                  

但是，一旦执行了session_destroy(),则在脚本结束，不会在执行写操作！      
 
**如何完全删除一个session？**                       
将于当前session相关的数据都删除掉。               
$_SESSION: unset，或者$_SESSION=array()       
session文件: session_destroy();                 
cookie中的sessionID: setcookie(‘PHPSESSID’, ‘’, time()-1);              


#### 服务器端session数据的处理
处理服务器上所保存的session数据的文件      
修改保存目录       
   
默认的保存在系统临时目录下，windows/temp下：          

可以通过修改 php的配置。session.save_path进行修改！注意，php不能自动创建该目录，应该由自己创建目录      

**分子目录保存session文件**        
每个session一个session文件，当请求变多时，session文件增多！ 分子目录保存！       

通过 修改  session.save_path完成！可以配置session的子目录级数，sessionid字符可能性等           

```
session.save_path="1;e:/php/temp"
session.hash_function=0
session.hash_bits_per_character=5
```

**利用数据库存储session**            

重写session的存储机制！


session面临的问题：    
1量大！难管理          
2服务器多，难共享！        

将session数据，放置在数据库服务器上进行保存管理！（内存）     

将session数据存入到数据库中！        

分析：        
需要的工作              
提供可以操作数据的功能代码！（读，写，删除）          
告知php，在需要的时候，调用我们所定义的代码！             

**增加六个session存储的处理函数（方法）**  

```
<?php

//定义六个函数

/**
 * 在session开启时执行，
 * 负责完成session存储所需要资源的初始化工作！
 */
function sess_open() {
	echo 'open<br>';
	//连接数据库
	$link = mysql_connect('127.0.0.1:3306', 'root', 'root');
	mysql_query('set names utf8');
	mysql_query('use itcast_shop');
}
/**
 * session_start()时，开启session时被执行
 *
 * 负责从当的session记录中，将session数据读取出来
 *
 * @param $sess_id string 当前的sessionID
 *
 * @return string session的数据，不需要序列化。如果没有读到则返回空字符串！
 */
function sess_read($sess_id) {
	echo 'read<br>';
	//利用 select 查询
	$sql = "select sess_data from `session` where sess_id='$sess_id'";
	$result = mysql_query($sql);
	if($row = mysql_fetch_assoc($result)) {
		return $row['sess_data'];
	} else {
		return '';
	}
}
/**
 * 在 脚本结束时被执行
 *
 * 负责，将当前的session数据，同步写到当前的session记录中
 *
 * @param $sess_id string
 * @param $sess_data string 
 *
 * @return bool
 */
function sess_write($sess_id, $sess_data) {
	echo 'write<br>';
	//执行写数据的操作！
	//当前session记录存在则更新sess_data，不存在则插入！
	$expire = time();
	$sql = "insert into `session` values ('$sess_id', '$sess_data', $expire) on duplicate key update sess_data='$sess_data', expire=$expire";
	return mysql_query($sql);
}

/**
 * 在调用了 session_destroy()系统函数时，被自动调用!\
 * 负责的功能时，利用当前id，删除当前的session记录！
 *
 * @param $sess_id string
 *
 * @return bool
 */
function sess_destroy($sess_id) {
	echo 'destroy<br>';
	//执行 delete 操作
	$sql = "delete from `session` where sess_id='$sess_id'";
	return mysql_query($sql);
}

/**
 * 在session_start() 是执行
 * 负责 删除所有的垃圾数据
 *
 * @param $maxlife int 最大的生命周期
 *
 * @return bool
 */
function sess_gc($ttl) {
	echo 'gc<br>';
	//
	$now = time();
	$last = $now-$ttl;//
	$sql = "delete from `session` where expire < $last";
	return mysql_query($sql);

}
/**
 * 脚本结束
 */
function sess_close() {
	echo 'close<br>';
//	mysql_close();
	return true;
}


session_set_save_handler(
	'sess_open',
	'sess_close',
	'sess_read',
	'sess_write',
	'sess_destroy',
	'sess_gc'
);
```

利用系统函数，设置session存储所需要的处理器，为当前定义好的 这六个函数：
session_set_save_handler();


**增加一个表保存session数据**         
保存在文件中变成保存到数据表中！          
一个文件对应	一条记录                  
文件名			某个字段，sessionID              
文件内容		字段，当前session数据                

```
create table session(
sess_id varchar(32) primary key,
sess_data text,
expire int(11)
) charset=utf8;
```

**垃圾回收，gc**              
如果一条记录（一个文件）在多久之内，没有被使用过了，则认为该数据是垃圾！        
默认的时间，是1440s秒。24分钟！可以通过php的配置：session.gc_maxlifetime=1440           
什么时候执行删除的动作？             
在session_start()时，有一定的几率，执行垃圾回收！       
默认是 1/1000 千分之一！      
可以通过php修改：          

```
session.gc_probability=1;
session.gc_divisor=1000;
```

- session与cookie的区别与联系?                   
session 的 sessionID存在 cookie内！       
- session数据有效期？原因是什么？          
浏览器关闭。                          
原因是保存在浏览器端的sessionid丢失!                  
默认的sessionID的cookie变量是临时的cookie变量！            
- 如何持久化session？

持久化sessionID这个cookie变量！

使用函数：session_id函数得到当前的sessionID！          

```
setcookie('PHPSESSID',session_id(),time()+3600);
```

php还提供一个专门的修改sessionID的cookie变量的函数：            
session_set_cookie_params();设置该cookie变量的选项的           

在表示有效期的第一个参数，是时间周期，而不是时间戳！留意，应该在session-start()之前，就完成cookie选项的设置！
（类似于 session_set_save_handler一样）

```
session_set_cookie_params('3600');
session_start();
```

此外还应该保证，服务器端垃圾判定的时间要与session持久化的时间尽量一致！
session.gc_maxlifetime = 3600;

留意，少用！            

- cookie禁用session是否可用？          
session是基于cookie的，因此cookie禁用，session不应该可用！

如果非要用，也是可以的！
办法是：
url上传递sessionID！

此时，需要修改php的配置达到目的：         
1,开启对url上传递的支持！将仅仅使用cookie传递sessionid关闭！           

```
session.use_only_cookie=0;
```

2，增加url上参数

```
session.use_trans_id=1;
```         
此时php就在浏览器没有cookie使用的情况下，可以采用url进行sessionID的传递：

注意，php只会自动在html代码的a连接的地址上增加url的传递SessionID
php代码内的url不能自动增加！

tip：表单请求时，该参数如何传递！php会自动增加表单项隐藏域，name=”PHPSESSID” 值为=”session_ID()”














