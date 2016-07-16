---
layout: post
title: PHP之MVC项目实战(二)
date: 2016-7-16
categories: blog
tags: [PHP]
description: PHP
---

**本文主要包括以下内容**       

1. GD库图片操作           
2. 利用GD库实现验证码      

#### GD库图片操作      

```
<?php

$img = imagecreatetruecolor(500, 300);
//var_dumP($img);
//
//分配绿色
$green = imagecolorallocate($img, 0, 0xff, 0x0);
//var_dump($green);

//fill
$result = imagefill($img, 0, 0, $green);
//var_dump($result);

//导出
imagepng($img, './green.png');

//
imagedestroy($img);
```

**创建画布**          
imagecreatefromXXXX XXX表示格式：           
//打开，利用已有的图片创建画布资源！          
imagecreatefromjpeg           
imagecreatefrompng，从png格式创建画布        
imagecreatefromgif                    

**操作画布**        
利用一个个的工具函数，完成画布的处理的！              
选择颜色，分配颜色                  
如果需要使用某个颜色，在画布上操作，一定要先将颜色分配到画布上！         
利用函数：          
imagecolorallocate(画布，颜色).向画布上分配颜色     
颜色是RGB，红绿蓝，颜色需要三个参数，分别表示R,G，B的值        
颜色标识= imagecolorallocate(画布，R，G，B)           
每个颜色值，是一个整型！         
0-255十进制          
0x0 - 0xff 十六进制          


**填充画布**      
利用函数：imagefill完成填充    
imagefill(画布，填充位置X, 填充位置Y，颜色);     
将像素周围的连续的并且颜色相同的区域可以完成填充！     
填充位置使用填充点的坐标表示：      
图片位置的原点为 左上角！坐标为(0,0) 因此右下角的坐标是？（width-1,height-1 499,299）         

**将画布导出成图片**    
imageXXXX，XXX表示格式        
imagejpeg               
imagegif 导出成gif格式     
imagepng                                   
一个画布可以导出多次，而且是任意格式！         

imagepng(画布，保存文件); 

**销毁资源**       
imagedestroy();


#### 利用GD库实现验证码            

```
<?php
//require '111.php';

$rand_bg_file = './captcha/captcha_bg' . mt_rand(1, 5) . '.jpg';
//echo $rand_bg_file;
//创建画布
$img = imagecreatefromjpeg($rand_bg_file);

//绘制边框
$white = imagecolorallocate($img, 0xff, 0xff, 0xff);
//不填充矩形
imagerectangle($img, 0, 0, 144, 19, $white);

//写码值
//生成码值，随机的4个只包含大写字母，和数字的字符串！
$chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';//32 个字符
//随机取4个
$captcha_str = '';
for($i=1,$strlen=strlen($chars); $i<=4; ++$i) {
	$rand_key = mt_rand(0, $strlen-1);
	$captcha_str .= $chars[$rand_key];
}
//保存到session中
@session_start();
$_SESSION['captcha_code'] = $captcha_str;
//echo $captcha_str;
//写

//先确定颜色，白黑随机！
$black = imagecolorallocate($img, 0, 0, 0);
if(mt_rand(0, 1) == 1 ) {
	$str_color = $white;
} else {
	$str_color = $black;
}

imagestring($img, 5, 60, 3, $captcha_str, $str_color);

//保存
//imagejpeg($img, './captcha.jpg');//输出到文件内！
//告知浏览器，发送的是jpeg格式的图片
header('Content-Type: image/jpeg; charset=utf-8');

//echo 'itcast';
imagejpeg($img);//输出到浏览器端！

//销毁资源
imagedestroy($img);
```

**绘制边框**      
画一个不填充的矩形！      
利用函数                
imagerectangle()完成                                                      

imagerectangle(画布，左上角X，左上角Y，右下角X，右下角Y，笔触颜色);         
利用左上角，与右下角的坐标确定矩形范围！           


**写验证码**      

写到 画布上        
利用函数        
imagestring()           
imagestring(画布，字体大小，位置X，Y，字符串，颜色);       
其中imagestring典型的是使用内置字体！（不支持中文）。字体大小1-5.5最大！            

**导出，保存**        
imagejpeg();   

**将验证码展示到页面上**

典型的：                              
使用一个php文件，直接输一个图片内容！         
直接输出的请求的浏览器端         
imagejpeg(画布，保存文件)               
如果没有第二个参数，则是直接输出！             

 




