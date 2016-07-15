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

