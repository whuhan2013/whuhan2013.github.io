---
layout: post
title: 图像处理中的matlab使用
date: 2016-12-19
categories: blog
tags: [matlab]
description: matlab
---

**图像的矩阵表示**     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/p2.png)   

**图像的输入输出和显示**     
可以使用函数imread将图像读入MATLAB环境，imread的基本语法是：    
imread （’filename’｝         
此处，filename是含有图像文件全名的字符串（包括任何可用的扩展名）。例如语句    

```
f = imread('Fig0101.tif');
```

将图像fig0101读取到图像数组f中。注意，单引号（’）是用来界定filename文件名字符串的，而命令行结尾处的分号在MATLAB中用于禁止输出。假如命令行中未包括分号，MATLAB将显示这一命令行指定的运算结果。当在MATLAB命令行窗口中出现提示符（》）时，表明命令行的开始      

使用imshow函数在MATLAB桌面显示图像，imshow的基本语法是：imshow(f)   
其中f是图像数组     

图1-1显示了在屏幕上的输出。注意，图窗编号出现在最终得到的图窗的左上部。如果另一
幅图像q随后用imshow来显示，MATLAB就用新图像取代

```
》figure,imshow(g) 
使用imwrite函数将图像写入当前日录，imwrite的基本语法如下：   
imwrite(f，’filename') 
```   

**类和图像类型**     
虽然使用的是整数坐标， 但 MATLAB 中的像素值（亮度）并未限制为整数。 表 1-1 列出了 MATLAB 和图像处理工具箱为描述像素值而支持的各种类。 表中的前 8 项是数值型的数据类，第 9 项称为字符类， 最后一项称为逻辑类。
uint8 和 logical 类广泛用于图像处理， 当以 TIFF 或 JPEG 图像文件格式读取图像时，会用到这两个类。 这两个类用1个字节表示每个像素。某些科研数据源， 比如医学成像， 要求提供超出 uint8 的动态范围：针对此类数据， 会采用 uint16 和 int16 类。 这两个类为每个矩阵元素使用2 个字节。针对计算灰度的操作， 比如傅立叶变换（见第 3 章）， 使用 double 和single 浮点类。 双精度浮点数每个数组元素使用8 个宇节， 而单精度浮点数使用 4 个字节。尽管工具箱支持 int8 、 uint32 和 int32 类， 但在图像处理中并不常用    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/p3.png)   

#### 算子       
一般我们用字母M和N分别表示矩阵中的行与列。1x N
矩阵被称为行向量， M*1矩阵被称为列向量， 1*1矩阵则被称为标量。    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p1.png)  


