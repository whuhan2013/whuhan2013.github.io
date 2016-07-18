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
3. 文件上传     
4. 缩略图          
5. 水印

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

此时应该在输出图片内容到浏览器之前，告知浏览器，当前内容是二进制的图片内容！        
header();//           
header(‘Content-Type: text/html; charset=utf-8’);//告知浏览器发送的内容以utf8编码形式的文件html数据！        

此时，使用 img标签的src属性，请求一个生成图片的php程序即可显示图片！   


如果向浏览器发送的是图片，则如果有错误浏览器不会显示，会将报错信息当作图片内容看待，所以此时如果要调错，则此时
要把header(‘Content-Type:image/jpeg’)先注释！需要严格输出的时候，PHP的开头<?php顶格写，结尾
?>不用写，任何空格也是当作输出。


#### 文件上传

数据在存储或者传输时，存在两种编码：字节码，二进制码！     
普通的字符串上传到服务器需要字节编码！    
但是将文件上传的编码是： 二进制编码！       

但是，默认的，浏览器是不会处理二进制编码的！将所有的数据都当字节码字符串处理！     

因此上传文件的第一个工作：      
告知浏览器，当前表单内，有需要被二进制编码的数据！（编码的数据是由多种编码组成） 
利用表单的属性：          
enctype=”multipart/form-data”             

```
<form action="112.php" method="post" enctype="multipart/form-data">
<input type="hidden" name="MAX_FILE_SIZE" value="50000" />
商品名:<input type="text" name="goods_name" />
<br>
商品图片:<input type="file" name="goods_img" />
<br>
<input type="submit">

</form>
```

此时浏览器在碰到文件域时，就知道以二进制形式编码！       

此时，提交到服务器上表单内的所有数据（字符串，文件）

服务器的PHP代码来处理数据：        
对字符串的处理：保存$_POST变量内（内存中）             
对文件的处理：将接收到文件，保存到服务器系统的临时目录下。有效期，当前脚本结束！          

用户需要在php的脚本执行周期内，将临时文件 持久化（拷贝到别的地方）    
利用函数：              
move_uploaded_file();//移动已经上传的文件              
移动结果 = move_uploaded_file(临时文件，目标文件)        
上传的临时文件的信息，被保存到超全局数组变量$_FILES中！            
为每个上传的文件，整理5个信息：          

- name: 原始文件名
- type: 类型。image/jpeg text/html 网络上标识资源类型方法。称之为MIME（多用途internel邮件扩展类型）类型。
- tmp_name：临时文件名
- error：错误标号（0,123467）
- size:大小，字节Byte         

```
<?php

/**
 * 拿到一个上传文件的信息
 * 判断其合理和合法性，移动到指定目标
 *
 * @param $file array 包含了5个上传文件信息的数组
 *
 * @return 成功，目标文件名；失败：false
 */
function upload($file) {
	//判断是否有错误
	if($file['error'] != 0 ) {
		//文件上传错误
		switch ($file['error']) {
			case 1:
				echo '文件太大，超出了php.ini的限制';
			break;
			case 2:
				echo '文件太大，超出了表单内的MAX_FILE_SIZE的限制';
			break;
			case 3:
				echo '文件没有上传完';
			break;
			case 4:
				echo '没有上传文件';
			break;
			case 6:
			case 7:
				echo '临时文件夹错误';
			break;

		}
		return false;
	}
	//判断类型
	$allow_types = array('image/jpeg', 'image/png', 'image/gif');
	if(!in_array($file['type'], $allow_types)) {
		echo '类型不对';
		return false;
	}
	//判断大小
	$max_size = 100000;//100k
	if($file['size'] > $max_size) {
		echo '文件过大';
		return false;
	}

	
	//处于安全性的考虑，判断是否是一个真正的上传文件：
	if( !is_uploaded_file($file['tmp_name'])) {
		echo '上传文件可疑';
		return false;
	}

	//移动
	//通常都会为目标文件重启名字
	//原则是：不重名，没有特殊字符，有一定的意义！
	$dst_file = uniqid('upload_') . strrchr($file['name'], '.');
	if (move_uploaded_file($file['tmp_name'], $dst_file)) {
		//移动成功，上传完毕！
		return $dst_file;
	} else {
		//失败
		echo '移动失败';
		return false;
	}
}

$result = upload($_FILES['goods_img']);
echo '<hr>';
var_dump($result);
```        

**代码解释**                       
唯一字符串 = uniqid(前缀)函数可以得到一个唯一的字符串！并且可以指定前缀：                                                                            
strrchr()。找到某个子字符串的最后出现位置，从该位置开始，截取到字符串的最后！      

**错误类型**

- 文件太大，超过了php的对上传文件的最大限制           
可以php的配置文件中修改：               
PHP对所有post数据的大小也有限制：所有的表单数据加起来，不能超过！    

- 文件过大，超出了表单内的一个隐藏元素的限制：               
该元素要出现在文件上传域之前                         


**上传文件工具类**               


```
<?php

class UploadTool {

	private $upload_dir;//上传目录
	private $max_size;
	private $allow_types;

	private $error_info;

	public function __construct($dir='', $size=2000000, $types=array()) {
		$this->upload_dir = $dir;
		$this->max_size = $size;
		$this->allow_types = empty($types)?array('image/jpeg', 'image/png'):$types;
	}

	public function __set($p_name, $p_value) {
		if (in_array($p_name, array('upload_dir', 'max_size', 'allow_types'))) {
			$this->$p_name = $p_value;
		}
	}
	public function __get($p_name) {
		if ($p_name  == 'error_info') {
			return $this->$p_name;
		}
	}

	/**
	 * 拿到一个上传文件的信息
	 * 判断其合理和合法性，移动到指定目标
	 *
	 * @param $file array 包含了5个上传文件信息的数组
	 * @param $prefix string 生成文件的前缀
	 *
	 * @return 成功，目标文件名；失败：false
	 */
	function upload($file, $prefix='upload_') {
		//判断是否有错误
		if($file['error'] != 0 ) {
			//文件上传错误
			switch ($file['error']) {
				case 1:
					$this->error_info = '文件太大，超出了php.ini的限制';
				break;
				case 2:
					$this->error_info = '文件太大，超出了表单内的MAX_FILE_SIZE的限制';
				break;
				case 3:
					$this->error_info = '文件没有上传完';
				break;
				case 4:
					$this->error_info = '没有上传文件';
				break;
				case 6:
				case 7:
					$this->error_info = '临时文件夹错误';
				break;

			}
			return false;
		}
		//判断类型
		if(!in_array($file['type'], $this->allow_types)) {
			$this->error_info = '类型不对';
			return false;
		}
		//判断大小
		if($file['size'] > $this->max_size) {
			$this->error_info = '文件过大';
			return false;
		}

		
		//处于安全性的考虑，判断是否是一个真正的上传文件：
		if( !is_uploaded_file($file['tmp_name'])) {
			$this->error_info = '上传文件可疑';
			return false;
		}

		//移动
		//通常都会为目标文件重启名字
		//原则是：不重名，没有特殊字符，有一定的意义！
		$dst_file = uniqid($prefix) . strrchr($file['name'], '.');
		if (move_uploaded_file($file['tmp_name'], $this->upload_dir . $dst_file)) {
			//移动成功，上传完毕！
			return $dst_file;
		} else {
			//失败
			$this->error_info = '移动失败';
			return false;
		}
	}

}
```

**分子目录保存**      

目录的创建，mkdir()!     
mkdir(目录位置，是否递归创建);      
假设b不存在，则：       
mkdir(‘a/b/c’);失败          
mkdir(‘a/b/c’, true)；成功            
判断目录是否已经存在，is_dir()判断是否是一个目录！               
bool = is_dir(目录地址)          

**划分子目录的原则！**
典型的原则，依据上传时间完成子目录的创建！        
按照小时创建子目录！       
利用函数 date()可以格式化时间。将一个时间戳，以易于理解的形式展示！            
date(‘格式’, 时间戳=time())         

tip：strtotime()将一个日期格式变成时间戳！         

```
//形成子目录
		$sub_dir = date('YmdH');
		//判断是否存在
		if(! is_dir($this->upload_dir . $sub_dir)) {
			//不存在则创建
			mkdir ($this->upload_dir . $sub_dir);
		}
```

**注意：**这里会报date(): It is not safe to rely on the system's timezone settings.       
**解决方法：**[解决方法](http://very1024.blog.51cto.com/3588520/1243649)



#### 缩略图                  

**基本过程**                                        
采样：在原图上采样（采集需要被复制的图像区域）         
拷贝：将采集的图样信息，拷贝到某个画布上（缩略图）         
修改：修改大小，修改拷贝之后的大小区域结果          

需要一个php的gd函数：            
imagecopyresampled();采样拷贝，并修改大小

需要的参数有哪些？           
两个画布：原图画布，与缩略图画布，$dst_img, $src_img         
采集区域控制：在原图上的一个矩形区域！左上角坐标，宽，高！        
拷贝到缩略图画布上的位置与大小：在缩略图画布上矩形！坐标和宽高！          

目标在前，而原图在后！先画布，左上角坐标，宽高！        
imagecopyresampled($dst_img, $src_img, $dst_x, $dst_y, $src_x, $src_y, $dst_w, $dst_h, $src_w, $src_h);

**实现步骤**          
1创建原图画布和缩略图画布        
2采样，拷贝，修改大小          
imagecopryresampled();             
3导出       
4销毁          

```
<?php

//创建原图画布和缩略图画布
$src_file = './src.jpg';
$src_img = imagecreatefromjpeg($src_file);//基于已有图片创建
//缩略图的大小应该如何确定？
$dst_img = imagecreatetruecolor(100, 100);//创建一个新的画布

//采样，拷贝，修改大小
imagecopyresampled($dst_img, $src_img, 0, 0, 0, 0, 100, 100, 500, 375);

//导出
header('Content-Type:image/jpeg');
imagejpeg($dst_img);

//销毁

imagedestroy($dst_img);
imagedestroy($src_img);
```

**等比例的缩略图**         
实现方式：                   
一种：限定缩略图的最大尺寸！                      
二种：补白生成大小一致的缩略图！              

```
<?php

//已知条件
$max_w = 100;//范围的最大的宽
$max_h = 100;//范围的最大的高
$src_file = './src.jpg';//原始图片

//计算原图的宽高
$src_info = getimagesize($src_file);
$src_w = $src_info[0];//原图宽
$src_h = $src_info[1];//原图高


//比较 宽之比 与 高之比
if($src_w/$max_w > $src_h/$max_h) {
	//宽应该缩放的多
	$dst_w = $max_w;//缩略图的宽为范围的宽
	$dst_h = $src_h/$src_w * $dst_w;//按照原图的宽高比将求出缩略图高
} else {
	$dst_h = $max_h;
	$dst_w = $src_w/$src_h * $dst_h;
}

//创建画布
$src_img = imagecreatefromjpeg($src_file);//基于已有图片创建
//缩略图的大小应该如何确定？
$dst_img = imagecreatetruecolor($dst_w, $dst_h);//创建一个新的画布

//采样，拷贝，修改大小
imagecopyresampled($dst_img, $src_img, 0, 0, 0, 0, $dst_w, $dst_h, $src_w, $src_h);

//导出
header('Content-Type:image/jpeg');
imagejpeg($dst_img);

//销毁
imagedestroy($dst_img);
imagedestroy($src_img);
```

**图片工具类**   

```
<?php


class ImageTool {

	/**
	 * 生成缩略图，补白
	 *
	 * @param $src_file
	 * @param $max_w;
	 * @param $max_h;
	 * 
	 * @return 缩略图的图片地址。失败false！
	 */
	public function makeThumb($src_file, $max_w, $max_h) {
		//判断原图片是否存在
		if (! file_exists($src_file)) {
			$this->error_info = '源文件不存在';
			return false;
		}

		//计算原图的宽高
		$src_info = getimagesize($src_file);
		$src_w = $src_info[0];//原图宽
		$src_h = $src_info[1];//原图高

		//在增加一个判断！
		//如果原图尺寸小于范围（缩略图尺寸）
		if($src_w < $max_w && $src_h < $max_h) {
			//则不用判断，直接用原图的
			$dst_w = $src_w;
			$dst_h = $src_h;
		} else {
			//比较 宽之比 与 高之比
			if($src_w/$max_w > $src_h/$max_h) {
				//宽应该缩放的多
				$dst_w = $max_w;//缩略图的宽为范围的宽
				$dst_h = $src_h/$src_w * $dst_w;//按照原图的宽高比将求出缩略图高
			} else {
				$dst_h = $max_h;
				$dst_w = $src_w/$src_h * $dst_h;
			}
		}

		//创建画布
		$src_img = imagecreatefromjpeg($src_file);//基于已有图片创建
		//缩略图的大小一致！
		$dst_img = imagecreatetruecolor($max_w, $max_h);//创建一个新的画布

		//为缩略图确定颜色,蓝色
		$blue = imagecolorallocate($dst_img, 0x0, 0x0, 0xff);
		imagefill($dst_img, 0, 0, $blue);//填充

		//采样，拷贝，修改大小。注意放置的位置！
		$dst_x=($max_w-$dst_w)/2;
		$dst_y=($max_h-$dst_h)/2;
		imagecopyresampled($dst_img, $src_img, $dst_x, $dst_y, 0, 0, $dst_w, $dst_h, $src_w, $src_h);

		//导出
		//取得原始文件的路径与名字
		$src_dir = dirname($src_file);
		$src_basename = basename($src_file);
		$thumb_file=substr($src_basename, 0, strrpos($src_basename, '.')) . '_thumb' . strrchr($src_basename, '.');

		imagejpeg($dst_img, $src_dir . DS . $thumb_file);


		//销毁
		imagedestroy($dst_img);
		imagedestroy($src_img);
		
		//返回
		return basename($src_dir) . '/' . $thumb_file;
	}
}
```


#### 水印      
将一张图片 合并 到另一张图片上！

采样：从印章图片上采集作为水印的图片！      
拷贝：将采集到的颜色信息，拷贝到需要增加水印的图片上！         
合并：需要确定合并位置（不能确定合并的大小），同时需要确定透明度！             

**一个函数：**     
imagecopymerge()          
需要的参数：                        
两个画布：印章，待增加水印的画布        
采集的区域：坐标和宽高   
合并的位置：坐标         
合并的透明度：透明值          
共 9 个参数！                           
画布，坐标，宽高，透明度；目标在前！                                                                   
imagecopymerge($dst_img, $stamp_img, $dst_x, $dst_y, $stamp_x, $stamp_y, $stamp_w, $stamp_h, $pct )         

```
<?php

$dst_file = './dst.jpg';
$stamp_file = './stamp.jpg';

//画布
$dst_img = imagecreatefromjpeg($dst_file);
$stamp_img = imagecreatefromjpeg($stamp_file);

//合并，拷贝
imagecopymerge($dst_img, $stamp_img, 0, 0, 0, 0, 128, 52, 70);

//
header('Content-Type: image/jpeg');
imagejpeg($dst_img);

imagedestroy($dst_img);
imagedestroy($stamp_img);
```









