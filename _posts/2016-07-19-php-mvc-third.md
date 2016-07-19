---
layout: post
title: PHP之MVC项目实战(三)
date: 2016-7-19
categories: blog
tags: [PHP]
description: PHP
---

**本文主要包括以下内容**       

1. 标准错误错误处理       
2. http操作      
3. PDO   
4. 文件操作


### 标准错误错误处理        

PHP在语法层面上发生的错误         
两个过程：     

触发阶段（发生一个错误）       
处理阶段（如何处理该错误）           

#### 触发阶段

- 系统触发，php自己触发           
典型的都是由php的核心在执行或者编译php代码时，发现的错误，并触发该错误！

- 用户触发，自定义错误           
但是可以通过用户的php代码，手动触发一个错误！
利用函数 trigger_error();触发一个用户自定义的错误！     


#### 错误处理阶段
有三种典型的错误处理方法

**第一:报告错误信息**   
将错误信息获取后，输出到浏览器。         
因此，典型的一个错误信息应包含：     
级别，消息，发生的文件，行号：             

想要管理错误报告，需要基于错误的级别进行管理：    
php允许管理：         
1，是否报告。            
2，报告哪些级别的错误。     

利用php的配置：                       
display_erorrs:是否显示错误信息，布尔。        
error_reporting:报告的级别！                    
需要利用位运算，将所有需要报告的级别都设置上！       
典型的级别是：           
显示所有的错误：        


也可以通过脚本代码进行以上配置          

```
ini_set('error_reporting', E_ALL | E_STRICT);
ini_set('display_errors', 1);
```

**第二:错误日志**       
将错误的信息写到日志文件中！               
也是通过两个配置完成处理？      

log_errors:是否开启错误日志        
error_log: 错误文件位置            
可以同归ini_set进行配置！     

```
//错误日志
ini_set('error_log', 'e:/php1016/apache/htdocs/test/test.error.log');
ini_set('log_errors', 1);
```

error_reporting:针对 错误日志同样有效！

上面的两种都是php内置的错误方式！ 


**第三:自定义的错误处理**            
当触发错误时，php自身不处理，交由用户脚本代码进行处理！  

用户要提供一个方法(函数）来处理发生的错误：定义函数！              
告知php，一旦发生错误，由用户定义的函数处理！        
定义一个处理函数：        

设置成错误处理器：         
有参数，四个参数：         
级别，消息，文件，行号：       

```
set_error_handler('user_error_handler');
function user_error_handler($level, $msg, $file, $line) {
	switch($level) {
		case E_NOTICE:
		case E_USER_NOTICE:
			echo '写到公告栏中,看到了再解决即可<br>';
			break;
		case E_WARNING:
		case E_USER_WARNING:
			echo '发一封e-mail，明天加班解决<br>';
			break;
		case E_USER_ERROR:
			echo '发送一个信息，马上来解决<br>';
//		exit;
	}

//	return false;
}
```

用户定义的处理器一但设置，则系统的（报告，日志）就不可以使用了！
但是通过使 用户的错误处理器返回 false的形式！。此时错误处理继续交由系统的处理器来处理。

**级别管理**                 
每个标准错误都有一个级别：                                   
在php中，是采用位运算的形式，管理各个标准的错误级别！             
可以在php代码中通过，php的预定义常量，看到，设置该级别：        
例如：                                                 
系统触发错误典型级别：E_NOTICE, E_WARNING,E_ERROR                              
用户触发错误的典型级别：E_USER_NOTICE, E_USER_WARNING, E_USER_ERROR         

一个典型的是E_ALL，表示所有的错误级别：                 

**操作**                           
ini_set(‘error_reporting’, 2047);                 
开启所有级别的错误：2047 二进制后，所有的位都为1！           

生产环境与开发环境典型错误配置：                                       
生产：开启所有的错误级别，不显示错误信息在页面上，而记录在日志内！                   
开发：开启所有级别，显示在页面上。关闭错误日志！                   

项目中增加一个配置项，表示当前的环境模型：            
dev，pro               
增加一个配置项：                
app/config/app.config.php          

然后在framework中初始化

```
private static function initErrorHandler() {
		if('dev' == $GLOBALS['config']['app']['run_mode']) {
			ini_set('error_reporting', E_ALL | E_STRICT);
			ini_set('display_errors', 1);
			ini_set('log_errors', 0);
		} elseif ('pro' == $GLOBALS['config']['app']['run_mode']) {
			ini_set('display_errors', 0);
			ini_set('error_log', APP_DIR . 'error.log');
			ini_set('log_errors', 1);
		}

	}
```

**致命错误的处理**      
会终止脚本运行！

但是如果是用户定义了错误处理器，并且触发的是用户的error级别，是可以恢复的（继续执行）    

但是，系统触发的致命错误，是不能被自定义的处理器所处理的！

致命错误都会终止脚本执行么？
不是，用户脚本触发的致命错误在，用户自定义了错误处理器时，可以恢复，脚本继续运行！



#### http操作

**PHP模拟GET请求**         

```
<?php

$host_ip = '127.0.0.1';
$port = '80';
if (!$link = fsockopen($host_ip, $port)) {
	//连接失败
	die('连接失败');
}
//var_dump($link);

//构建get请求字符串数据
$request_str = 'GET /test.php HTTP/1.1' . "\r\n";//请求行
//请求头
$request_str .= 'Host: shop.100.com' . "\r\n";//请求主机
$request_str .= 'User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:25.0) Gecko/20100101 Firefox/25.0' . "\r\n";//请求代理标识，模拟的firefox
//请求头以空行结束
$request_str .= "\r\n";//空行
//没有请求主体
//发请求
$result = fwrite($link, $request_str);
//var_dump($result);


//等待接收响应数据
echo '<pre>';
echo fread($link, 500);
//while (!feof($link)) {
//	echo fgets($link);
//}
```

**模拟POST请求**  

```
<?php

$host_ip = '127.0.0.1';
$port = '80';
if (!$link = fsockopen($host_ip, $port)) {
	//连接失败
	die('连接失败');
}

//post数据
$admin_name = 'admin';
$admin_pass = '1234abcd';
$post_data = 'username='.$admin_name . '&password='.$admin_pass;
//username=admin&password=1234abcd

//构建post请求字符串数据
$request_str = 'POST /index.php?p=back&c=Admin&a=signin HTTP/1.1' . "\r\n";//请求行
//请求头
$request_str .= 'Host: shop.100.com' . "\r\n";//请求主机
$request_str .= 'User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:25.0) Gecko/20100101 Firefox/25.0' . "\r\n";//请求代理标识，模拟的firefox
//post数据相关的请求头
$request_str .= 'Content-Type: application/x-www-form-urlencoded' . "\r\n";//post主体数据类型
$request_str .= 'Content-Length: ' . strlen($post_data) . "\r\n";
//请求头以空行结束
$request_str .= "\r\n";//空行
//请求主体部分
$request_str .= $post_data;//不需要\r\n

//发送！
fwrite($link, $request_str);

//获得相应数据
echo '<pre>';
//echo fread($link, 8000);
while (!feof($link)) {
	echo fgets($link);
}
```

**http下载**                                   
下载，将服务器端的数据，保存到浏览器端！

http下载，就是将原本应该在浏览器上打开的数据数据，让浏览器保存起来即可！

我们服务器需要工作：              
1，将需要下载的内容输出到浏览器！                   
2，告知浏览器，接收到的响应数据应该保存起来！、                
利用一个响应头，Content-disposition: 告知浏览器内容的处理方法：          
Content-disposition: attachment 将响应数据作为附件来对待！            

http下载，只是下载的响应主体！        


```
<?php
//test.php得到文件的标识，id
//利用id，获得文件信息。
$img_file = 'E:\php1016\apache\htdocs\shop\app\upload\2013111909\goods_528abce32871b.png';

header('Content-Disposition: attachment; filename=' . basename($img_file));
$finfo = finfo_open(FILEINFO_MIME);//获得一个可以得到文件的MIME信息的资源
$mime = finfo_file($finfo, $img_file);//利用这个资源获得文件的信息
header('Content-Type: ' . $mime);


readfile($img_file);
```

**curl：client URL**             
php的专门用于模拟请求的扩展！            
（curl，独立工具，php支持curl库而已）           

使用之前开启扩展

```
<?php

$curl = curl_init();
//var_dump($curl);

curl_setopt($curl, CURLOPT_URL, 'http://shop.100.com/index.php?p=back&c=Admin&a=signin');
curl_setopt($curl, CURLOPT_POST, true);//false
curl_setopt($curl, CURLOPT_POSTFIELDS, array('username'=>'admin', 'password'=>'1234abcd'));
curl_setopt($curl, CURLOPT_HEADER, true);
//curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_COOKIEJAR, 'e:/php1016/apache/htdocs/test/curl_cookie.txt');
echo '<pre>';
curl_exec($curl);

curl_close($curl);
```

**控制缓存（浏览器的缓存**          
控制的浏览器端的缓存

典型的响应头，Expires                 
告知浏览器，当前你收到的响应数据应该的有效期到什么时候！

header()函数完成响应头的设置：

注意，此时的时间应该是一个格林威治平时          
	Tue, 19 Nov 2013 07:27:14 GMT        
因此，Expires也是GMT时间：          

gmdate();与date功能一致，将一个时间戳格式化成一个时间！date格式成本地时间（带时区的）。而gmdate格式化成一个GMT时间！


```
<?php

header('Expires: ' . gmdate('D, d M Y H:i:s', time()-1) . ' GMT');
header('Cache-Control: no-store no-cache must-revalidate');
echo gmdate('Y-m-d H:i:s');
echo '<br>';
echo gmdate('D, d M Y H:i:s') . ' GMT';
echo '<br>';
echo date('Y-m-d H:i:s');
echo '<hr>';
?>

<a href="121.php">self</a>
```

**防止盗链**

利用 请求时携带的请求头：      
referer：表示当前请求的来源！   

只要在请求图片的响应，判断一下当前的请求是否是来源本站即可！

得到来源。判断来源！

可以在Apache上，或者 php上都可以！

对于图片的请求都是用php生成！

通过判断$_SERVER[‘HTTP_REFERER’] 是否是当前网站！


#### PDO

另一种php操作mysql（或其他数据库）方法

PHP Data Object
语法上 是 oop，对象编程（类，方法，属性，对象）

PDO 是一个 数据库抽象层，PDO可以完成对大多数主流数据的操作

pdo操作具体的数据库，至少要：         
开启PDO，同时需要开启相应的驱动！        
PDO已经内置，mysql的驱动没有内置！     

```
extension=php_pdo_mysql.dll
```

**简单使用**      

```
/**
* @abstract PDO study: PDO操作MySQL增删查改类
* @date  2014/06/30
* @author Silov[bluebird237@gmail.com]
*/

class mysqlPdoClass{
	private $db;
	private $db_name = 'study';
	private $db_serv;
	private $db_user = 'root';
	private $db_pass = 'root';
	private $db_host = 'localhost';
	//构造函数以及连接数据库
	public function __construct( $username = 'root' , $password = 'root' , $connect = 'false'){
		$this->db_serv = "mysql:dbname=".$this->db_name.";host=".$this->db_host.";charset=utf-8";
		$this->db_user = $username;
		$this->db_pass = $password;
	}
	//管理数据库，析构函数
	public function __destruct(){
		$this->db = null;
	}
	//执行SQL语句，返回值为sql执行结果
	public function run_query($sql){
		$this->db = new PDO($this->db_serv, $this->db_user, $this->db_pass);
		$this->db->query("set names utf8");	//设置PHP+MySQL连接的编码格式为UTF8
		$row = $this->db->query($sql);
		$row->setFetchMode(PDO::FETCH_ASSOC);	//设置查询结果显示为键值对数组模式
		return $row;
	}
	//执行SQL语句，返回值为sql影响行数
	public function run_exec($sql){
		$this->db = new PDO($this->db_serv, $this->db_user, $this->db_pass);
		$this->db->query("set names utf8");    //设置PHP+MySQL连接的编码格式为UTF8
		return $this->db->exec($sql);
	}
	/**
	* @abstract 插入操作
	* @param 	$table:表名; $data:插入数据键值对数组; $return:是否返回值,为true时,返回插入字段的id; $debug:是否测试,true时返回sql语句,不执行
	* @return 	返回值:插入结果，boolean
	*/
	public function data_insert($table, $data, $return = true,$debug=false){
		if(!$table) {
			return false;
		}
		$fields = array();
		$values = array();
		foreach ($data as $field => $value){
			$fields[] = '`'.$field.'`';
			$values[] = "'".addslashes($value)."'";
		}
		if(empty($fields) || empty($values)) {
			return false;
		}
		$sql = 'INSERT INTO `'.$table.'` 
				('.join(',',$fields).') 
				VALUES ('.join(',',$values).')';
		if($debug){
			return $sql;
		}
		$query = $this->run_exec($sql);
		return $return ? $this->db->lastInsertId() : $query;
	}
	/**
	* @abstract 更新操作
	* @param 	$table:表名; $condition:更新查询条件; $data:更新数据键值对数组; $limit:更新数据条数上限
	*			$debug:测试时给true，则不执行update，直接返回完整的sql语句
	* @return 	返回值:插入结果，boolean
	*/
	public function data_update($table, $condition, $data, $limit = 1,$debug=false) {
		if(!$table) {
			return false;
		}
		$set = array();

		foreach ($data as $field => $value) {
			$set[] = '`'.$field.'`='."'".addslashes($value)."'";
		}
		if(empty($set)) {
			return false;
		}
		$sql = 'UPDATE `'.$table.'` 
				SET '.join(',',$set).' 
				WHERE '.$condition.' '.
				($limit ? 'LIMIT '.$limit : '');
		if($debug){
			return $sql;
		}
		return $this->run_exec($sql);
	}
	/**
	* @abstract 查询单个字段值
	* @param 	$sql:sql语句
	* @return 	返回值:string
	*/
	public function getOne($sql){
		$row = $this->run_query($sql);
		$data = $row->fetch();
		$data = array_shift($data);
		return $data;
	}
	/**
	* @abstract 查询单条记录，多个字段
	* @param 	$sql:sql语句
	* @return 	返回值:键值对数组
	*/
	public function getRow($sql){
		$row = $this->run_query($sql);
		$data = $row->fetch();
		return $data;
	}
	/**
	* @abstract 查询多条记录
	* @param 	$sql:sql语句
	* @return 	返回值:以键值对数组为元素的数组
	*/
	public function getRows($sql){
		$row = $this->run_query($sql);
		$data = $row->fetchAll();
		return $data;
	}
}
```

**预处理的SQL的执行方式**
如果需要重复地执行结构相同的SQL。此时结构相同的SQL意味着SQL的编译结果是类似的。除掉数据部分，在做重复的工作！
应该如何解决？           
将SQL中结构相同的先提取，编译。     
将不一样的地方，独立的处理。       
最后执行将数绑定了的结果可以            

```
<?php
$dsn = 'mysql:dbname=itcast_shop;host=127.0.0.1;port=3306';
$username = 'root';
$password = '123456';
$pdo = new PDO($dsn, $username, $password);


//$sql = "insert into it_admin (admin_id, admin_name, admin_pass) values (null, 'itcast', md5('1234abcd'))";
//$rows = $pdo->exec($sql);
//$sql = "insert into it_admin (admin_id, admin_name, admin_pass) values (null, 'php', md5('1234abcd'))";
//$rows = $pdo->exec($sql);
//$sql = "insert into it_admin (admin_id, admin_name, admin_pass) values (null, '1016', md5('1234abcd'))";
//$rows = $pdo->exec($sql);
//$sql = "insert into it_admin (admin_id, admin_name, admin_pass) values (null, 'han', md5('1234abcd'))";
//$rows = $pdo->exec($sql);

$sql_struct = "insert into it_admin (admin_id, admin_name, admin_pass) values (null, :name, :pass)";

$stmt = $pdo->prepare($sql_struct);

$data  = array(
	array('java', '1234'),
	array('php', '12345'),
	array('.net', '4567')
);
foreach($data as $row) {
	$stmt->bindValue(':name', $row[0]);
	$stmt->bindValue(':pass', md5($row[1]));

	$stmt->execute();
}
```

**错误的处理**              
mysql的扩展，错误的处理方式不会主动报错，称之为 静默模式。           

pdo的错误的处理：默认的也是 静默模式！    
此时需要使用 错误函数获得错误信息：           
$pdo->errorCode();//获得错误代码      
$pdo->errorInfo();//错误信息集合             

pdo还支持其他的错误模式：        
需要通过修改PDO对象的错误模式属性进行修改：    
$pdo->setAttribute()方法可以完成属性的设置     
$pdo->setAttribute(属性名，属性值);        

**静默模式：Silent mode**        
默认的  

**警告模式：Warning Mode**        
一旦发生错误，则触发一个警告级别的错误！ 

**异常模式：Exception mode**         
异常，类似于标准错误，是属于php在处理oop语法时，新的一种错误的处理模式！

分成三个阶段进行处理：       
抛出，监听，捕获！        
使用如下语法实现：             
抛出：throw           
监听：try         
捕获：catch       

抛出的异常，其实就是将所有的错误信息封装到一个异常对象中！           
异常：语法上就是一个 内置的Exception类（其扩展类也算）的对象！                   

典型的语法如下：          

```
<?php

$dsn = 'mysql:dbname=itcast_shop;host=127.0.0.1;port=3306';
$username = 'root';
$password = '123456';
$pdo = new PDO($dsn, $username, $password);
//设置错误模式：
//$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT);
//$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_WARNING);
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

echo '<pre>';
try {
	$sql = 'show database';
	if (!$pdo->query($sql)) {
		echo $pdo->errorCode();
		echo '<br>';
		var_dump($pdo->errorInfo());
	}
} catch (PDOException $e) {
	echo $e->getMessage();
}
```


#### 文件操作            

```
<?
class FSC{

// 函数名: getfilesource
// 功能: 得到指定文件的内容
// 参数: $file 目标文件
// test passed
function getfilesource($file){
    if($fp=fopen($file,'r')){
        $filesource=fread($fp,filesize($file));
        fclose($fp);
        return $filesource;
    }
    else
        return false;
}
// 函数名: writefile
// 功能: 创建新文件，并写入内容，如果指定文件名已存在，那将直接覆盖
// 参数: $file -- 新文件名
// $source  文件内容
//test passed
function writefile($file,$source){
    if($fp=fopen($file,'w')){
        $filesource=fwrite($fp,$source);
        fclose($fp);
        return $filesource;
    }
    else
        return false;
}
// 函数名: movefile
// 功能: 移动文件
// 参数: $file -- 待移动的文件名
// $destfile -- 目标文件名
// $overwrite 如果目标文件存在，是否覆盖.默认是覆盖.
// $bak 是否保留原文件 默认是不保留即删除原文件
// test passed
function movefile($file,$destfile,$overwrite=1,$bak=0){
    if(file_exists($destfile)){
        if($overwrite)
            unlink($destfile);
        else
            return false;
    }
    if($cf=copy($file,$destfile)){
        if(!$bak)
            return(unlink($file));
        }
    return($cf);
}
  
// 函数名: movedir
// 功能: 这是下一涵数move的附助函数，功能就是移动目录
function movedir($dir,$destdir,$overwrite=1,$bak=0){
     @set_time_limit(600);
    if(!file_exists($destdir))
        FSC::notfate_any_mkdir($destdir);
    if(file_exists($dir)&&(is_dir($dir)))
        {
        if(substr($dir,-1)!='/')$dir.='/';
        if(file_exists($destdir)&&(is_dir($destdir))){
        if(substr($destdir,-1)!='/')$destdir.='/';
            $h=opendir($dir);
            while($file=readdir($h)){
                if($file=='.'||$file=='..')
                    {
                    continue;
                    $file="";
                }
                if(is_dir($dir.$file)){
                    if(!file_exists($destdir.$file))
                        FSC::notfate_mkdir($destdir.$file);
                    else
                        chmod($destdir.$file,0777);
                    FSC::movedir($dir.$file,$destdir.$file,$overwrite,$bak);
                    FSC::delforder($dir.$file);
                    }
                else
                {
                    if(file_exists($destdir.$file)){
                        if($overwrite)unlink($destdir.$file);
                        else{
                            continue;
                            $file="";
                            }
                    }
                    if(copy($dir.$file,$destdir.$file))
                        if(!$bak)
                            if(file_exists($dir.$file)&&is_file($dir.$file))
                                @unlink($dir.$file);
                }
            }
        }
        else
            return false;
    }
    else
        return false;
}
// 函数名: move
// 功能: 移动文件或目录
// 参数: $file -- 源文件/目录
//       $path -- 目标路径
//       $overwrite -- 如是目标路径中已存在该文件时，是否覆盖移动
//                  --  默认值是1, 即覆盖
//       $bak  -- 是否保留备份(原文件/目录)
function move($file,$path,$overwrite=1,$bak=0)
     {
    if(file_exists($file)){
        if(is_dir($file)){
            if(substr($file,-1)=='/')$dirname=basename(substr($file,0,strlen($file)-1));
            else $dirname=basename($file);
            if(substr($path,-1)!='/')$path.='/';
            if($file!='.'||$file!='..'||$file!='../'||$file!='./')$path.=$dirname;
            FSC::movedir($file,$path,$overwrite,$bak);
            if(!$bak)FSC::delforder($file);
            }
        else{
            if(file_exists($path)){
                if(is_dir($path))chmod($path,0777);
                else {
                    if($overwrite)
                        @unlink($path);
                    else
                        return false;
                }
            }
            else
                FSC::notfate_any_mkdir($path);
            if(substr($path,-1)!='/')$path.='/';
            FSC::movefile($file,$path.basename($file),$overwrite,$bak);
        }
    }
    else
        return false;
}
// 函数名: delforder
// 功能: 删除目录,不管该目录下是否有文件或子目录，全部删除哦，小心别删错了哦!
// 参数: $file -- 源文件/目录
//test passed
function delforder($file) {
     chmod($file,0777);
     if (is_dir($file)) {
          $handle = opendir($file);
          while($filename = readdir($handle)) {
           if ($filename != "." && $filename != "..")
            {
                FSC::delforder($file."/".$filename);
            }
          }
          closedir($handle);
          return(rmdir($file));
     }
     else {
        unlink($file);
      }
}
// 函数名: notfate_mkdir
// 功能: 创建新目录,这是来自php.net的一段代码.弥补mkdir的不足.
// 参数: $dir -- 目录名

function notfate_mkdir($dir,$mode=0777){
    $u=umask(0);
    $r=mkdir($dir,$mode);
    umask($u);
    return $r;
}
// 函数名: notfate_any_mkdir
// 功能: 创建新目录,与上面的notfate_mkdir有点不同，因为它多了一个any,即可以创建多级目录
//         如:notfate_any_mkdir("abc/abc/abc/abc/abc")
// 参数: $dirs -- 目录名

function notfate_any_mkdir($dirs,$mode=0777)
{
  if(!strrpos($dirs,'/'))
    {
      return(FSC::notfate_mkdir($dirs,$mode));
  }else
      {
      $forder=explode('/',$dirs);
      $f='';
      for($n=0;$n<count($forder);$n++)
          {
          if($forder[$n]=='') continue;
          $f.=((($n==0)&&($forder[$n]<>''))?(''):('/')).$forder[$n];
          if(file_exists($f)){
              chmod($f,0777);
              continue;
              }
          else
              {
              if(FSC::notfate_mkdir($f,$mode)) continue;
              else
                  return false;
          }
        }
        return true;
      }
}
}
?>
```