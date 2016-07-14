---
layout: post
title: PHP数据库操作
date: 2016-7-14
categories: blog
tags: [数据库]
description: PHP数据库操作 
---


**PHP实现数据库的增删改查**       

```
<?php
$conn=mysql_connect('localhost','root','root');
if(!$conn){
echo "connect failed";
exit;
}

$sql='use test';
mysql_query($sql,$conn);

//增加
$sql="insert into mytest values(null,'pu','20')";
$rs=mysql_query($sql,$conn);
if($rs){
  echo 'insert data success'."<br />";
}else{
  echo 'insert data failed'."<br />";
}

//修改
$sql="update mytest set longtitude='fu' where id=3";
$rs=mysql_query($sql,$conn);
if($rs){
  echo 'update data success'."<br />";
}else{
  echo 'update data failed'."<br />";
}

//删除
$sql="delete from mytest where id='2'";
$rs=mysql_query($sql,$conn);
if($rs){
  echo 'del data success'."<br />";
}else {
  echo 'del data failed'."<br />";  
}

//查询
$sql="select * from mytest";
$rs=mysql_query($sql,$conn);
//打印变量的相关信息
var_dump($rs);
echo "<br/>";
//遍历从结果集中取行 mysql_fetch_array/assoc/row/object
while($row=mysql_fetch_object($rs)){
print_r($row);
echo '   ';
echo $row->id;
echo "<br />";
}
?>
```

**effect**   

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p4.png)


#### php连接mysql的基本操作  

- 连接认证

```
header('Content-Type: text/html; charset=utf-8');
$host = '127.0.0.1';
$port = '3306';
$user = 'root';
$pass = 'root';
$charset = 'utf8';
$link = mysql_connect("$host:$port", $user, $pass);
if(!$link)
{
  die('连接失败');
}
```

- 向mysql发送sql       
mysql_query(sql, 连接资源);        
失败返回false，成功返回资源或者true！          
可以使用 mysql _error(连接) mysql_errno(连接)获得错误信息和标识      

- 执行sql，生成结果（mysql-server）                
执行成功后：返回数据可以是资源也可以true。执行失败一定是false！          
依据所执行的 sql，是否有返回数据！           
返回资源：有返回数据：select，show，desc。      
返回true：没有返回数据的： use，set，insert，update，delete，DDL        

- 处理结果         
称之为结果集（result set）类型资源！

结果集：结果的集合！

将数据，从结果集中取出来！称之为 fetch！
使用函数：           
mysql_fetch_assoc|row|array。功能完全一致，只是返回的数据格式不同！

在结果集中，取得一条记录。结果集内也存在结果集记录指针的概念！
fetch一次，只能取得当前记录，但是可以向后移动记录指针！配合上循环结构可以将所有的记录从结果集中取出！

- 关闭连接
mysql_free_result(结果集)          
mysql_close(连接资源);                 

