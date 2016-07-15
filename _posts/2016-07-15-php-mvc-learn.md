---
layout: post
title: PHP之MVC学习
date: 2016-7-15
categories: blog
tags: [PHP]
description: PHP
---

**代码架构进货过程**    

#### one，混编        
嵌入式脚本语言PHP       
html与php混编的编码方式            

#### two，显示和逻辑相分离

最后，需要将显示和逻辑的结果放在一起！
需要在 php页面，将html代码 载入才可以！

```
<?php
// 业务逻辑部分
// 得到数据

//得到所有的比赛信息
mysql_connect('127.0.0.1:3306', 'root', '123456');
mysql_query('set names utf8');
mysql_query('use itcast');
$sql = "select p1.id as p1_id, p2.id as p2_id, m.match_time, p1.stu_name as p1_name, m.match_result, p2.stu_name as p2_name from select_match as m left join select_student as p1 on m.player_1=p1.id left join select_student as p2 on m.player_2=p2.id";
$result = mysql_query($sql);

while($row=mysql_fetch_assoc($result)) {
	$rows[] = $row;
}

//载入 负责显示的html代码
require './template/match_list.html';
```

**模板文件**            
注意当前的html页面，出现：显示格式部分由html代码充当，而数据部分需要通过php来实现。数据，特指数据的展示。（数据的获得实在负责逻辑的php代码中完成）
这样的html文件，就被称之为模（mu）板文件（template）

**限制用户访问php逻辑文件**        
办法多得是，典型的两种： 

1，.htaccess
deny from all                
利用Apache的对访问的控制，将某个目录设置成禁止访问。将所有的模板都放入到目录内！           
注意，htaccess生效的前提是，Apache对目录开启了allowoverride :

```
<Directory "D:\amp\apache\htdocs\test">
     DirectoryIndex index.php
     AllowOverride All
     Allow from all     
</Directory>
```

2，将不允许用户访问的文件，包含模板文件，都放置文档根目录之外！            
浏览器请求只能看到文档根目录下的文件。


#### three，处理数据（处理业务逻辑的代码），从php代码中分离
即MVC架构           

mvc，项目的分层思想，指的是完成一个业务逻辑，需要三大部分，分别是：           
1，具体的业务逻辑实现的部分，数据操作，称之为 M，Model，模型！         
2，具体显示样式的实现部分，html+css+js，称之为 V，View，视图！             
3，负责整体流程控制的部分，负责调用M和V。成为 C，Controller，控制器！        


**模型类**     
模型的处理最终都是要处理数据库的，建立一个数据库工具类        

```
<?php
/**
 * mysql数据操作类
 */
class MySQLDB {
	//属性
	//对象的初始化属性
	private $host;
	private $port;
	private $user;
	private $pass;
	private $charset;
	private $dbname;

	//运行时生成的属性
	private $link;
	private $last_sql;//最后执行的SQL

	private static $instance;//当前的实例对象

	/**
	 * 构造方法
	 * @access private
	 *
	 * @param $params array 对象的选项
	 */
	private function __construct($params = array()) {
		
		//初始化 属性
		$this->host = isset($params['host']) ? $params['host'] : '127.0.0.1';
		$this->port = isset($params['port']) ? $params['port'] : '3306';
		$this->user = isset($params['user']) ? $params['user'] : 'root';
		$this->pass = isset($params['pass']) ? $params['pass'] : '';
		$this->charset = isset($params['charset']) ? $params['charset'] : 'utf8';
		$this->dbname = isset($params['dbname']) ? $params['dbname'] : '';

		//连接数据库
		$this->connect();
		//设置字符集
		$this->setCharset();
		//设置默认数据库
		$this->selectDB();
		
	}
	/**
	 * 克隆
	 * @access private
	 */
	private function __clone() {
	}
	/**
	 * 获得单例对象
	 */
	public static function getInstance($params) {
		if (! (self::$instance instanceof self) ) {
			//实例化时，需要将参数传递到构造方法内
			self::$instance = new self($params);
		}
		return self::$instance;
	}

	/**
	 * 连接数据库
	 */
	private function connect() {
		if(!$link = mysql_connect("$this->host:$this->port", $this->user, $this->pass)) {//$this->host . ':' . $this->port
			echo '连接失败，请检查mysql服务器，与用户信息';
			die;
		} else {
			//连接成功，记录连接资源
			$this->link = $link;
		}
	}

	/**
	 * 设置字符集
	 */
	private function setCharset() {
		$sql = "set names $this->charset";
		return $this->query($sql);
	}

	/**
	 * 设置默认数据库
	 */
	private function selectDB() {
		//判断是否存在一个数据库名
		if($this->dbname === '') {
			return ;
		}

		$sql = "use `$this->dbname`";
		return $this->query($sql);
	}

	/**
	 * 执行SQL的方法,PHPDocumentor
	 *
	 * @param $sql string 待执行的SQL
	 *
	 * @return mixed 成功返回 资源 或者 true，失败，返回false
	 */
	public function query($sql) {
		$this->last_sql = $sql;
		//执行，并返回结果
		if(!$result = mysql_query($sql, $this->link)) {
			echo 'SQL执行失败<br>';
			echo '出错了SQL是：', $sql, '<br>';
			echo '错误代码是：', mysql_errno($this->link), '<br>';
			echo '错误信息是：', mysql_error($this->link), '<br>';
			die;
			return false;//象征性的！
		} else {
			return $result;
		}
	}

	/**
	 * @param $sql string 待执行的sql
	 * @return array 二维
	 */
	public function fetchAll($sql) {
		//执行
		if ($result = $this->query($sql)) {
			//成功
			//遍历所有数据，形成一个二维数组
			$rows = array();//初始化
			while($row = mysql_fetch_assoc($result)) {
				$rows[] = $row;
			}
			//释放结果集
			mysql_free_result($result);
			return $rows;	
		} else {
			//执行失败
			return false;
		}
	}

	/**
	 * 执行SQL，获得符合条件的第一条记录
	 *
	 * @param $sql string 待执行的SQL
	 *
	 * @return array 一维数组
	 */
	public function fetchRow($sql) {
		if ($result = $this->query($sql)) {
			$row = mysql_fetch_assoc($result);
			mysql_free_result($result);
			return $row;
		} else {
			return false;
		}
	}

	/**
	 * 利用一个SQL，返回符合条件的第一条记录的第一个字段的值
	 *
	 * @param $sql string 待执行的SQL
	 *
	 * @return string 执行结果
	 */
	public function fetchColumn($sql) {
		if ($result = $this->query($sql) ) {
			if ($row = mysql_fetch_row($result)) {//row返回的是索引数组，因此0元素，一定是第一列
				mysql_free_result($result);
				return $row[0];
			} else {
				return false;
			}
		} else {
			return false;
		}
	}
			

	
	/**
	 * 在序列化时被调用
	 *
	 * 用于负责指明哪些属性需要被序列化
	 *
	 * @return array
	 */
	public function __sleep() {
		return array('host', 'port', 'user', 'pass', 'charset', 'dbname');
	}
	
	/**
	 * 在反序列化时调用
	 *
	 * 用于 对对象的属性进行初始化
	 */
	public function __wakeup() {
		//连接数据库
		$this->connect();
		//设置字符集
		$this->setCharset();
		//设置默认数据库
		$this->selectDB();
	}
	
}
```

建立一个model父类处理数据 

```
<?php

/**
 * 模型的基础类
 */
class Model {
	protected $db;//保存MySQLDB类的对象

	/**
	 * 构造方法
	 */
	public function __construct() {
		//连接数据库
		$this->initLink();
	}

	/**
	 * 初始化数据库的连接
	 */
	protected function initLink() {
		require './MySQLDB.class.php';
		$options = array(
			'host'=>'127.0.0.1',
			'port'=>'3306',
			'user'=>'root',
			'pass'=>'123456',
			'charset'=>'utf8',
			'dbname'=>'test'
		);
		$this->db = MySQLDB::getInstance($options);

//		mysql_connect('127.0.0.1:3306', 'root', '123456');
//		mysql_query('set names utf8');
//		mysql_query('use itcast');
	}
}
```

子类继承它并得到相应数据    

```
<?php

require_once './Model.class.php';

class ClassModel extends Model {

	public function getList() {
//		mysql_connect('127.0.0.1:3306', 'root', '123456');
//		mysql_query('set names utf8');
//		mysql_query('use itcast');
		$sql = "select c.*, count(s.id) as s_count from select_class as c left join select_student as s on c.class_id=s.class_id group by c.class_id";
		return $this->db->fetchAll($sql);
//		$result = mysql_query($sql);
//		while($row=mysql_fetch_assoc($result)) {
//			$rows[] = $row;
//		}
//		return $rows;
	}
}
```

在Controller中调用即可  

```
<?php

//调用模型获得数据
require './ClassModel.class.php';
$model_class = new ClassModel;
$rows = $model_class->getList();

//调用视图显示数据
require './template/class_view.html';
```


#### 控制器的处理        
每增加一个功能，都要增加一个控制器，加入模块的概念，表示操作的集合。        

```
<?php

// 班级管理模块
// 所有班级的功能，全都放在该文件中完成！

$action = isset($_GET['a']) ? $_GET['a'] : 'list';

if ('list' == $action) {
	//功能一：班级列表操作
	//调用模型获得数据
	require './ClassModel.class.php';
	$model_class = new ClassModel;
	$rows = $model_class->getList();
	//调用视图显示数据
	require './template/class_view.html';
}
elseif ('del' == $action) {
	//功能二：班级删除功能
	//删除班级的控制器
	//得到需要删除的ID
	//要利用模型，将相应的数据删除
	require './ClassModel.class.php';
	$model_class = new ClassModel;
	$model_class->delClass($_GET['id']);
	//不需要视图，直接返回列表页面即可
	header('Location: class_controller.php');
}
elseif ('N' == $action) {
//功能N：
}

else {

	header('Location: class_module.php');
}
```


**控制器类**       
将刚刚所完成一个控制器模块文件，升级成一个控制器类

```
<?php

class ClassController {

	public function listAction() {
		//调用模型获得数据
		require './ClassModel.class.php';
		$model_class = new ClassModel;
		$rows = $model_class->getList();
		//调用视图显示数据
		require './template/class_view.html';
	}

	public function delAction() {
		require './ClassModel.class.php';
		$model_class = new ClassModel;
		$model_class->delClass($_GET['id']);
		//不需要视图，直接跳转返回列表页面即可
		header('Location: index.php?a=list');
	}
}
```

**利用index.php作为程序的入口**  

```
<?php

//确定当前的控制器类
$c = isset($_GET['c']) ? $_GET['c'] : 'Class';
$controller_name = $c . 'Controller';
//实例化控制器类，调用Action方法执行
//require './ClassController.class.php';
require './' . $controller_name . '.class.php';
$controller = new $controller_name;//可变类名的语法

//调用？
$action = isset($_GET['a']) ? $_GET['a'] : 'list';
$action_name = $action . 'Action';
$controller->$action_name();//利用了可变函数
```



