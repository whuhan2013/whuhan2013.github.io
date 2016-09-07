---
layout: post
title: PHP之校花评比项目 
date: 2016-9-7
categories: blog
tags: [PHP]
description: 校花项目
---

**本文主要用到了以下内容**  

1. 前端页面布局         
2. 使用 Jquery 控制页面效果和操作 Cookie
3. 使用 Ajax 方式与后台交互
4. PHP 操作 MySQL 完成数据查询
5. PHP 实现埃洛等级分系统算法


**开启服务器**  

```
$ php -S localhost:8080        #端口可自定义
将localhost改成0.0.0.0可以在公网访问
```


### 项目制作

2.1 项目简介

**评选过程：**


> 介绍： 校花排名页面随机产生两个女生(名字做了处理) ， 比较两个女生的颜值， 然后利用鼠标点击选择一个你认为较漂亮的女生，随后数据自动提交到数据后台处理， 页面会自动刷新，再次随机产生两个女生，再次选择比较， 依次达到不断更新颜值的作用。 上方进度条可以显示你的进度，当进行10次选择以后， 页面自动弹到排名页面， 显示美女排名 ，以及校花。

想法来自《社交网络》，所以这里教程的步骤也跟着扎克伯格的脚步来进行。


**排名算法**

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p11.png)


#### 数据库

首先按照上面提供的方法开启数据库服务。进入并创建数据库：

```
$ mysql -u root
$ mysql> CREATE DATABASE beauty DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
$ mysql> use beauty;
```

设计 stu 数据表：

数据库存储女生信息.（id，姓名，图片，颜值）

```
CREATE TABLE `stu` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `stu` varchar(255) DEFAULT NULL,
  `img` varchar(255) DEFAULT NULL,
  `score` int(20) NOT NULL DEFAULT '1400',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=78 DEFAULT CHARSET=utf8;
SET FOREIGN_KEY_CHECKS=1;
```

填充部分示例数据：

```
INSERT INTO `stu` VALUES ('1', 'Mary', 'imgs/1.jpg', '1400'),
 ('2', 'Nancy', 'imgs/2.jpg', '1400'),
 ('3', 'Kacy', 'imgs/3.jpg', '1400'),
 ('4', 'Judegli', 'imgs/4.jpg', '1400'),
 ('5', 'Kacy1', 'imgs/7.jpg', '1400'),
 ('6', 'Judegli1', 'imgs/6.jpg', '1400');
```


#### 前端布局

前端设计采用 bootstrap 响应式布局，适合任何设备使用。主要布局如下(具体看下载下来的页面代码)：

index.php ：

```
<?php 
require_once 'DBMysql.php';
$db = DBMysql::connect();
$sql = 'select * from stu order by rand() limit 2';
$result = $db->query($sql);
while ($row = $result->fetch_assoc()) {
    $results[] = $row;
}
 ?>

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Campus campus Belle ranking selection</title>
    <link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="//cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <style type="text/css" media="screen">
        body {
          padding-top: 50px;
        }
        .main {
          padding: 40px 15px;
          text-align: center;
        }
        .footer {
          position: absolute;
          bottom: 0;
          width: 100%;
          /* Set the fixed height of the footer here */
          height: 60px;
          background-color: #191E29;
          text-align: center;
          color: #DFDFDF;
        }
    </style>
</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <a class="navbar-brand" href="http://www.shiyanlou.com">Campus Belle ranking (shiyanlou.com)</a>
      </div>
    </nav>
    <div class="container">
      <div class="main">

        <div class="alert alert-info alert-dismissible" role="alert" hidden="">
          <button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>
          <strong>!</strong>Of the two girls, who do you think is more beautiful~
        </div>

        <div class="progress">
            <div class="progress-bar" role="progressbar" aria-valuenow="60" aria-valuemin="0" aria-valuemax="100" style="">
            <span class="">0% Complete</span>
            </div>
        </div>
        
        <div class="row">
            <?php 
                foreach ($results as $key => $value) {
             ?>
            <div class="col-sm-6 col-md-6 rankimg<?php echo $key; ?>">
                <input type="hidden" name="" value="<?php echo $value['id'] ?>">
                <a href="#" class="thumbnail" id="rank<?php echo $key; ?>">
                    <img src=" <?php echo $value['img'] ?> " alt="..." class="img-rounded">
                </a>
                <span><a href="#" title=""><?php echo $value['stu'] ?></a></span>
            </div>
            <?php } ?>
        </div>
      </div>
    </div><!-- /.container -->
    <footer class="footer">
      <div class="container">
        <p class="text-muted"><h4>Campus campus Belle ranking selection</h4><h5>www.shiyanlou.com</h5></p>
      </div>
    </footer>
    <script src="//cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script>
        $(document).ready(function() {
            function getCookie(name) {
                var arr,reg=new RegExp("(^| )"+name+"=([^;]*)(;|$)");
                if(arr=document.cookie.match(reg)){
                    return (arr[2]);
                } else {
                    return null;
                }
            }
            if(!getCookie("rankwoman"))
            {
                $(".alert-info").show();
                document.cookie="rankwoman=rankwoman";
                document.cookie="rankwomanper=0";
            }
            var rankwomanper=parseInt(getCookie("rankwomanper"));
            $('#rank0').click(function() {
                $('#rank1').hide();
                rankwomanper=parseInt(getCookie("rankwomanper"))+1;
                document.cookie="rankwomanper="+rankwomanper;
                rank(0);
            });
            $('#rank1').click(function() {
                rankwomanper=parseInt(getCookie("rankwomanper"))+1;
                document.cookie="rankwomanper="+rankwomanper;
                $('#rank0').hide();
                rank(1);
            });
            function rank(i) {
                $.post('./Rank.php', {stu1: $('.rankimg0 input').val(),stu2:$('.rankimg1 input').val(),vid:i}, function(data, textStatus, xhr) {
                    if (textStatus == 'success') {
                        window.location.reload();
                    }
                });
            }
            if(getCookie("rankwomanper")>9){
                window.location.href="./ranklist.php";
            } else {
                $(".progress-bar").width(rankwomanper+"0%");
                $(".progress-bar span").text(rankwomanper+"0% Competed");
            }
        });
    </script>
</body>
</html>
```

同理，在排名列表中，我们也需要做类似的处理展示：

ranklist.php :

```
<?php 
require_once 'DBMysql.php';
$db = DBMysql::connect();
$sql = "SELECT * FROM `stu` ORDER BY `score` DESC";
$result = $db->query($sql);
while ($row = $result->fetch_assoc()) {
    $results[] = $row;
}
 ?>
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Campus campus Belle ranking selection</title>
    <link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css">
    <script src="//cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <style type="text/css" media="screen">
        body {
          padding-top: 50px;
        }
        .main {
          padding: 40px 15px;
          text-align: center;
        }
    </style>
</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <a class="navbar-brand" href="http://www.shiyanlou.com">Campus Belle ranking (shiyanlou.com)</a>
      </div>
    </nav>
    <div class="container">        
        <table class="table table-hover">
          <caption>美女颜值排行表</caption>
          <thead>
            <tr>
              <th>Section</th>
              <th>Name</th>
              <th>RankScore</th>
            </tr>
          </thead>
          <tbody>
          <?php foreach ($results as $key => $value) {  ?>
            <tr>
              <th scope="row"><?php echo $key+1 ?></th>
              <td><a href="#" title=""><?php echo $value['stu'] ?></a></td>
              <td><?php echo $value['score'] ?></td>
            </tr>
            <?php } ?>
          </tbody>
        </table>
        <button class="btn btn-success" id="rechoose" style="float: right">Choose Again</button>
    </div><!-- /.container -->
    <script src="//cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script type="text/javascript">
    $("#rechoose").on("click",function(){
       document.cookie="rankwomanper=0";
       window.location.href="./index.php";
    });
    </script>
    </body>
</html>
```


#### PHP 后台处理

后台主要分为两个部分：数据库部分和颜值计算部分。

**数据库操作辅助类**

使用此类可以很方便的操作数据库。

DBMysql.php ：

```
<?php

class DBMysql
{
	public static function connect(){
		$dbc = new mysqli('localhost','root','','beauty') OR die('Could not connected to MySQL: '.mysql_error());
		$dbc->query('SET NAMES utf8');
		return $dbc;
	}
}
```

定义了一个数据库连接方法，使用 mysqli 建立mysql 连接，设置字符编码，返回连接对象。

**颜值计算类**

此类根据埃洛等级分系统设计算法，实现读取、计算、更新颜值的操作。


```
<?php 
/**
* 排名算法
*/
class Rank
{
	private static $K = 32;
	private static $db;
	function __construct()
	{
		require_once 'DBMysql.php';
		self::$db = DBMysql::connect();
	}

	public function queryScore($id)
	{
		$sql = "SELECT * FROM stu WHERE `id` = $id";
		$info = mysqli_fetch_assoc(self::$db->query($sql));
		return $info['score'];
	}

	public function updateScore($Ra,$id)
	{
		self::$db->query("UPDATE `stu` SET `score` = $Ra WHERE `id` = $id");
	}

	public function expect($Ra,$Rb)
	{
		$Ea = 1/(1+pow(10,($Rb-$Rb)/400));
		return $Ea;
	}

	public function calculateScore($Ra,$Ea,$num)
	{
		$Ra = $Ra + self::$K*($num-$Ea);
		return $Ra;
	}

	public function selectStu()
	{
		$id1 = $_POST['stu1'];
		$id2 = $_POST['stu2'];
		$victoryid = $_POST['vid'];
		return $this->getScore($id1,$id2,$victoryid);
	}

	public function getScore($id1,$id2,$victoryid)
	{
		$Ra = $this->queryScore($id1);
		$Rb = $this->queryScore($id2);
		if ($Ra & $Rb) {
			$Ea = $this->expect($Ra, $Rb);
			$Eb = $this->expect($Rb, $Ra);
			$Ra = $this->calculateScore($Ra, $Ea, 1-$victoryid);
			$Rb = $this->calculateScore($Rb, $Eb, $victoryid);
			$Rab = array($Ra,$Rb);
			$this->updateScore($Ra, $id1);
			$this->updateScore($Rb, $id2);
			return $Rab;
		} else {
			return false;
		}
	}
}
$Rank = new Rank();
$Rank->selectStu();
```

代码示例讲解：若前端页面展示的两位女生的 id 为 3 和 7，若用户选择 3，则 js 向后台传递的数据为 3,7,0，后台处理 女生 a 的 id 为3，女生 b 的 id 为7，获胜的 id 为 0 ，表示第一个获胜，则本轮真实得分 ：a 的得分为 1，b 的得分为 0。计算调整后分数时：$Ra = $this->calculateScore($Ra, $Ea, 1); $Rb = $this->calculateScore($Rb, $Eb, 0);，代码中使用 1-$victoryid 和 $victoryid 来处理。

现在进入 Beauty 目录，开启 PHP 内置服务器，打开浏览器就可以看到效果了。


**源码地址**    

[校花排名项目](https://github.com/whuhan2013/pythoncode/tree/master/Beauty)

**访问地址：**[Campus campus Belle ranking selection](http://119.29.152.242:8888/)

**效果如下**

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p9.png)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p10.png)