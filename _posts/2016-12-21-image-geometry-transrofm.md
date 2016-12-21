---
layout: post
title: 图像的几何变换
date: 2016-12-21
categories: blog
tags: [图像处理]
description: 图像的几何变换
---

包含相同内容的两幅图像可能由于成像角度、透视关系乃至镜头自身原因所造成的几何失
真而呈现出截然不同的外观，这就给观测者或是图像识别程序带来了困扰。通过适当的几何变
换可以最大程度地消除这些几何失真所产生的负面影响，有利于我们在后续的处理和识别工作
中将注意力集中子图像内容本身，更确切地说是图像中的对象，而不是该对象的角度和位置等。
因此， 几何变换常常作为其他图像处理应用的预处理步骤， 是图像归一化的核心工作之一。  

**本文主要包括以下内容**    

- 图像的各种几何变换， 如平移、镜像和旋转等
- 插值算法
- 图像配准
- 本章的典型案例分析
  + 基于直方图均衡化的图像灰度归一化
  + 汽车牌照的投影失真校正

#### 解决几何变换的一般思路      
图像几何变换又称为图像空间变换， 它将一幅图像中的坐标位置映射到另一幅图像中的
新坐标位置． 我们学习几何变换的关键就是要确定这种空间映射关系， 以及映射过程中的变
换参数。       
几何变换不改变图像的像素值， 只是在图像平面上进行像素的重新安排。一个几何变换
需要两部分运算：首先是空问变换所需的运算， 如平移、旋转和镜像等， 需要用它来表示输
出图像与输入图像之间的〈像素〉映射关系：此外， 还需要使用灰度插值算法， 因为按照这
种变换关系进行计算， 输出图像的像素可能被映射到输入图像的非整数坐标上。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p1.png)  
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p2.png)  

之所以要逆变换是因为:   
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p3.png)  


#### 图像平移  
图像平移就是将图像中所有的点按照指定的平移量水平或者垂直移动。     

**图像平移的变换公式**    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p4.png)  

对于原图中被移出图像显示区域的点通常也有两种处理方法：直接丢弃或者通过加目标图像的尺寸〈将新生成的图像宽度增加$T_x$，高度增加$T_y$〉的方法使新图像中能够包含这些点，在稍后给出的程序实现中，我们来用了第一种处理方法。．  

```
init = imread('lena.bmp'); % 读取图像
[R, C] = size(init); % 获取图像大小
res = zeros(R, C); % 构造结果矩阵。每个像素点默认初始化为0（黑色）
delX = 50; % 平移量X
delY = 50; % 平移量Y
tras = [1 0 delX; 0 1 delY; 0 0 1]; % 平移的变换矩阵 

for i = 1 : R
    for j = 1 : C
        temp = [i; j; 1];
        temp = tras * temp; % 矩阵乘法
        x = temp(1, 1);
        y = temp(2, 1);
        % 变换后的位置判断是否越界
        if (x <= R) & (y <= C) & (x >= 1) & (y >= 1)
            res(x, y) = init(i, j);
        end
    end
end;

figure;
subplot(1,2,1);
imshow(uint8(init)); title('原图像'); 
subplot(1,2,2);
imshow(uint8(res)); % 显示图像
title('平移后');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p5.png)  


#### 图像镜像
镜像变换又分为水平镜像和坚直镜像。水平镜像即将图像左半部分和右半部分以图像坚直中轴线为中心轴进行对换；而竖直镜像则是将图像上半部分和下半部分以图像水平中轴线为中心轴进行对换.   

**图像镜像的交换公式**     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p6.png)  

**matlab实现**     
imtransform函数用于完成一般的二维空间变换，形式如下:      
B=imtransform(A,TFrom,method)    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p7.png)  

```
A=imread('lena.bmp');
[height,width,dim]=size(A);
tform= maketform('affine',[-1 0 0;0 1 0;width 0 1]);

B = imtransform(A,tform,'nearest');
tform2=maketform('affine',[1 0 0;0 -1 0;0 height 1]);
C = imtransform(A,tform2,'nearest');

subplot(1,3,1),imshow(A);
title('原图像');
subplot(1,3,2),imshow(B);
title('水平镜像');
subplot(1,3,3),imshow(C);
title('竖直镜像');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter4/p8.png)  



