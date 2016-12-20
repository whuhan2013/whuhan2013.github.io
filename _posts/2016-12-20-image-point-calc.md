---
layout: post
title: 图像的点运算
date: 2016-12-20
categories: blog
tags: [图像处理]
description: 图像处理
---

对于一个数字图像处理系统来说， 一般可以将处理流程分为3个阶段。在获取原始图像
后， 首先是图像预处理阶段， 其次是特征抽取阶段，最后才是识别分析阶段。预处理阶段尤
为重要， 这个阶段处理不好则直接导致后面的工作无法展开。        
点运算指的是对图像中的每个像素依次进行同样的灰度变换运算。设r和s分别是输入
图像f(x,y）和输出图像g(x,y）在任一点(x，y）的灰度值，则点运算可以使用下式定义    
$$s=T(r)$$          
其中， T为采用的点运算算子， 表示在原始图像和输出图像之间的某种灰度级映射
关系。
点运算常常用于改变图像的灰度范围及分布， 是图像数字化及图像显示时常常需要的工具． 点运算因其作用的性质有时也被称为对比度 增强、对比度拉伸或灰度变换。     

**本章主要包括以下内容**       

- 最基本的图像分析工具:灰度直方图
- 利用直方图辅助实现的各种灰度变换， 包括灰度线性变换、伽马变换、阀值变换和窗
口变换等
- 两种实用的直方图修正技术:直方图均衡化和直方图规定化
- 本章的典型案例分析
	+ 基于直方图均衡化的图像灰度归一化
	+ 直方图匹配

#### 灰度直方图     
灰度直方圈描述了一幅图像的灰度级统计信息，主要应用于图像分割和图像灰度变换等
处理过程中．      

**理论基础**      
从数学角度来说，图像直方图描述图像各个灰度级的统计特性，它是图像灰度值的函数，
统计一幅图像中各个灰度级出现的次数或概率。有一种特殊的直方图叫做归一化直方图， 可
以直接反映不同灰皮级出现的比率。     
从图形上来说， 灰度直方图是一个二维图， 横坐标为图像中各个像素点的灰度级别， 纵坐标表示具有各个灰度级别的像素在图像中出现的次数或概率。      

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter3/p1.png)  

**matlab实现**       
Matlab中的imhist函数可以进行图像的灰度直方圈运算，调用语法为：

```
imhist(I)
imhist(I,n)
[count,x]=imhist(...)
```

- I为需要计算灰度直方图的图像．   
• n为指定的灰度级数目． 如果指定参数n，则会将所有的灰度级均匀分布在n 个小区
间内，而不是将所有的灰度级全部分开．        
• counts为直方图数据向量. counts(i）表示第i个灰度区间中的像素数目．
• x是保存了对应的灰度小区间的向量．     

若调用时不接收这个函数的返回值，贝Jj直接显示直方图：在得到这些返回数据之后，也
可以使用stem(x, count)来手工绘制直方图。    

**一般直方图**     

```
I = imread('pout.tif');
figure;
imshow(I); title('Souce');
figure;
imhist(I);title('Graph');
figure;
imhist(I,64);
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter3/p2.png)  

**归一化直方图**    

在由imhist 函数的返回值中， counts 保存了落入每个区间的像素的个数， 通过计算 couts与图像中像素总数的商可以得到归一化的直方图。

```
I = imread('pout.tif');
figure;
[M,N]=size(I);
[counts,x]=imhist(I,32);
counts= counts / M/N;
stem(x,counts);
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter3/p3.png) 

分析图像的灰度直方图往往可以得到很多有效的信息。 例如， 从图 3.4的一系列灰度直 方图上， 可以很直观地看出图像的亮度和对比度特征。 实际上， 直方图的峰值位置说明了图 像总体上的亮暗： 如果图像较亮， 则直方图的峰值出现在直方图的较右：部分：如果图像较暗．则直方图的峰值出现在直方图的较左部分， 从而造成暗部细节难以分辨。 如果直方图中只有中间某一小段非零值，贝IJ这张图像的对比度较低： 反之， 如果直方图的非零值分布很宽而且比较均匀， 则图像的对比度较高。

#### 灰度的线性变换     
线性灰度变换函数f(x）是一个一维线性函数：      
$$D_B = f(D_A) ＝f_AD_A +f_B$$     
式中参数$f_A$为线性函数的斜率， $f_B$为线性函数在y轴的截距， $D_A$ 表示输入图像的灰度，
$D_B$ 表示输出图像的灰度。      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter3/p4.png) 

**matlab实现**      
使用Matlab对图像执行线性变换无需专门的函数， 下面的程序对Matlab示例图像 coiilsJpg进行了不同参数的线性变换操作

```
I = imread('coins.png');		% 读入原图像

I = im2double(I);			% 转换数据类型为double
[M,N] = size(I);			% 计算图像面积

figure(1);				% 打开新窗口
imshow(I);				% 显示原图像
title('原图像');

figure(2);				% 打开新窗口
[H,x] = imhist(I, 64);		% 计算64个小区间的灰度直方图
stem(x, (H/M/N), '.');		% 显示原图像的直方图
title('原图像');

% 增加对比度
Fa = 2; Fb = -55;
O = Fa .* I + Fb/255;

figure(3);
subplot(2,2,1);
imshow(O);
title('Fa = 2 Fb = -55 增加对比度');

figure(4);
subplot(2,2,1);
[H,x] = imhist(O, 64);
stem(x, (H/M/N), '.');
title('Fa = 2 Fb = -55 增加对比度');

% 减小对比度
Fa = 0.5; Fb = -55;
O = Fa .* I + Fb/255;

figure(3);
subplot(2,2,2);
imshow(O);
title('Fa = 0.5 Fb = -55 减小对比度');

figure(4);
subplot(2,2,2);
[H,x] = imhist(O, 64);
stem(x, (H/M/N), '.');
title('Fa = 0.5 Fb = -55 减小对比度');

% 线性增加亮度
Fa = 1; Fb = 55;
O = Fa .* I + Fb/255;

figure(3);
subplot(2,2,3);
imshow(O);
title('Fa = 1 Fb = 55 线性平移增加亮度');

figure(4);
subplot(2,2,3);
[H,x] = imhist(O, 64);
stem(x, (H/M/N), '.');
title('Fa = 1 Fb = 55 线性平移增加亮度');

% 反相显示
Fa = -1; Fb = 255;
O = Fa .* I + Fb/255;

figure(3);
subplot(2,2,4);
imshow(O);
title('Fa = -1 Fb = 255 反相显示');

figure(4);
subplot(2,2,4);
[H,x] = imhist(O, 64);
stem(x, (H/M/N), '.');
title('Fa = -1 Fb = 255 反相显示');
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter3/p5.png) 

