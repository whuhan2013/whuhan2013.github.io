---
layout: post
title: 图像分割
date: 2017-1-1
categories: blog
tags: [图像处理]
description: 图像分割
---

图像分割是指将图像中具有特殊意义的不同区域划分开来， 这些区域互不相交，每个区域满足灰度、纹理、彩色等特征的某种相似性准则。图像分割是图像分析过程中最重要的步骤之一，分割出的区域可以作为后续特征提取的目标对象。

**本文主要包括以下内容**    

- 基于梯度的Sobel、Prewitt和Roberts算子的边缘检测
- LoG边缘检测算法
- Canny边缘检测算法
- Hough变换和直线检测
- 阙值分割技术
- 基于区域的图像分割技术
- 本章的典型案例
	+ 基于LoG和Canny算子的精确边缘检测
	+ 基于Hough变换的直线检测
	+ 图像的四叉树分解

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p1.png)

### 边缘检测     
图像的边缘是图像的最基本特征，边缘点是指图像中周围像素灰度有阶跃变化或屋顶变化的那些像素点，即灰度值导数较大或极大的地方。图像属性中的显著变化通常反映了属性 的重要意义和特征。        
边缘检测是图像处理和计算机视觉中的基本问题，其用途在于标识数字图像中亮度变化明显的点。我们曾在5.5节中讨论了一些可以用于增强边缘的图像锐化方法，本节介绍如何 将它们用于边缘检测。此外，还将介绍一种专门用千边缘检测的Canny算子  

#### 边缘检测概述
边缘检测可以大幅度地减少数据量， 并且剔除那些被认为不相关的信息， 保留图像重要的结构属性。         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p2.png)  

**边缘检测方法的分类**       
通常可将边缘检测的算法分为两类： 基于查找的算法和基于零穿越的算法。除此之外．还有Canny边缘检测算法、统计判别方法等。       

- 基于查找的方法是指通过寻找图像一阶导数中的最大和最小值来检测边界，通常将边界定位在梯度最大的方向， 是基于一阶导数的边缘检测算法．
- 基于零穿越的方法是指通过寻找图像二阶导数零穿越来寻找边界。通常是拉普拉斯过零点或者非线性差分表示的过零点，是基于二阶导数的边缘检测算法．  

基于—阶导数的边缘检测算子包括Roberts算子、Sobel算子、Prewitt算子等，它们都是梯度算子；基于二阶导数的边缘检测算子主要是高斯—拉普拉斯边缘检测算子.     

#### 常用的边缘检测算子       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p3.png)  
Roberts算子利用局部差分算子寻找边缘，边缘定位精度较高，但容易丢失一部分边缘，同时由于图像没经过平滑处理，因此不具备抑制噪声的能力。该算子对具有陡峭边缘且含噪声小的图像效果较好。          
Sobel算子和Prewitt算子都考虑了邻域信息，相当于对图像先做加权平滑处理，然后再做微分运算，所不同的是平滑部分的权值有些差异，因此对噪声具有一定的抑制能力，但不能完全排除检测结果中出现的虚假边缘。虽然这两个算子边缘定位效果不错，但检测出的边缘容易出现多像素宽度。      

**高斯一拉普拉斯算子**     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p4.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p5.png) 

**Canny边缘检测算子**         
前面介绍的几种都是基于微分方法的边缘检测算法，他们都只有在图像不含噪声或者首先通过平滑去除噪声的前提下才能正常应用。        
在图像边缘检测中，抑制噪声和边缘精确定位是无法同时满足的，一些边缘检测算法通过平滑滤波去除噪声的同时，也增加了边缘定位的不确定性；而提高边缘检测算子对边缘敏感性的同时，也提高了对噪声的敏感性。Canny算子力图在抗噪声干扰和精确定位之间寻求最佳折衷方案。         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p6.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p7.png) 
显然，只要固定了k, 就固定了极大值的个数。         
有了这3个准则，寻找最优的滤波器的问题就转化为泛函的约束优化问题了，公式的解可以由高斯的一阶导数去逼近。            
Canny边缘检测的基本思想就是首先对图像选择一定的Gauss滤波器进行平滑滤波，然后采用非极值抑制技术进行处理得到最后的边缘图像。其步骤为：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p8.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p9.png) 

#### matlab实现       
Matlab的IPT函数edge可以方便地实现9.2.2小节的几种边缘检测方法， 该函数的作用 是检测灰度图像中的边缘， 并返回一个带有边缘信息的二值图像， 其中黑色表示背景， 白色 表示原图像中的边缘部分.       

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p10.png)       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p11.png)

**基于高斯一拉普拉斯算子的边缘检测**      
使用edge函数进行基于高斯－拉普拉斯算子的边缘检测的调用语法为：
BW = edge(I,'log',thresh,sigma);         
第2个参数为'log'表示采用高斯拉普拉斯算子．        
thresh是敏感度阈值参数，任何灰度值低于此阈值的边缘将不会被检测到． 其默认值是空矩阵D, 此时算法会自动计算阈值． 如果将thresh设为0, 则输出的边缘图像将包含围绕所有物体的闭合的轮廓线， 因为这样的运算会包括输入图像中所有的过零点          
sigma指定生成高斯滤波器所使用的标准差．默认时，标准差值为2. 滤镜大小n ＊ n,n的计算方法为n=ceil(sigma ＊ 3) x 2+1.

**基干Canny算子的边缘检测**     
BW = edge(I,'canny',thresh,sigma);   
第2个参数为'canny'表示采用canny算子．        
thresh是敏感度阈值参数， 其默认值是空矩阵[]. 与前面的算法不同， Canny算法的敏感度阈值是一个列向量， 因为需要为算法指定阈值的上下限． 在指定阈值矩阵时，第1个元素是阈值下限，第2个元素为阈值上限．如果只指定一个阈值元素，则这个直接指定的值会被作为阈值上限，而它与0.4的积会被作为阈值下限。如果阈值参数没有指定， 算法会自行确定敏感度阈值的上、下限。          
sigma指定生成平滑使用的高斯滤波器的标准差．默认时，标准差值为1滤镜大小nxn, n的计算方法为n＝ceil(sigma x 3) x 2+1.

```
intensity = imread('circuit.tif');
bw1 = edge(intensity,'sobel');
bw2 = edge(intensity,'prewitt');
bw3 = edge(intensity,'roberts');
bw4 = edge(intensity,'log');
bw5 = edge(intensity,'canny');

subplot(3,2,1); imshow(intensity); title('a');
subplot(3,2,2); imshow(bw1); title('b');
subplot(3,2,3); imshow(bw2); title('c');
subplot(3,2,4); imshow(bw3); title('d');
subplot(3,2,5); imshow(bw4); title('e');
subplot(3,2,6); imshow(bw5); title('f');
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p12.png)
从图中可以看出， 不同算法得到的结果存在很大差异， 下面进行简要的分析         

- 从边缘定位的精度看       
Roberts算子和Log算子定位精度较高。       
Roberts算子简单直观，Log算子则利用二阶导数零交叉特性检测边缘。但Log算子只能获得边缘位置信息， 不能得到边缘的方向等信息。      
- 从对不同方向边缘的响应看       
从对边缘方向的敏感性而言，Sobel算子、Prewitt算子检测斜向阶跃边缘效果较好，Roberts算子检测水平和垂直边缘效果较好。Log算子则不具备边缘方向检测能力。而Sobel算子可以提供最精确的边缘方向估计。          
- 从去噪能力看
Roberts和Log算子定位精度虽然较高， 但受噪声影响大。      
Sobel算子和Prewitt算子模板相对较大因而去噪能力较强， 具有平滑作用， 能滤除一些噪声， 去掉部分伪边缘， 但同时也平滑了真正的边缘， 这也正是其定位精度不高的原因。      
从总体效果来衡量， Canny算子给出了一种边缘定位精确性和抗噪声干扰性的较好折衷办法。

**注意**：以上验证结杲及分析是基于阶跃变化假设进行的，但真实的灰度变化不一定都是阶跃的，有可能发生在很宽的灰度范围上， 且存在灰度的起落． 因此，我们应当根据工程实际对各种算子做以比较后加以选用．  

### 霍夫变换      
9.2节介绍了一些边缘检测的有效方法。但实际中由于噪声和光照不均等因素，使得在很多情况下获得的边缘点不连续，必须通过边缘连接将它们转换为有意义的边缘。一般的做 法是对经过边缘检测的图像进一步使用连接技术，从而将边缘像素组合成完整的边缘。          
霍夫(Hough)变换是一个非常重要的检测间断点边界形状的方法。它通过将图像坐标空间变换到参数空间，来实现直线和曲线的拟合。    

#### 直线检测      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p13.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p14.png)     
注意：由于原图中的直线往往具有一定宽度，实际上相当于多条参数极其接近的单像素宽直线，它们对应参数空间中相邻的多个累加器单元． 因此每找到一个当前最大的峰值点后，需要将该点及其附近点清零，以防算法检测出多条极其邻近的“假”直线．     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter9/p15.png)  
这种利用二维累加器的离散化方法大大简化了Hough变换的计算.参数空间a-b上的细分程度决定了最终找到直线上点的共线精度。上述的二维累加数组A组也常常被称为Hough 矩阵。  

**注意**：使用直角坐标表示直线， 当直线为一条垂直直线或者接近垂直直线时，该直线的斜率为无限大或者接近无限大，从而无法在参数空间a-b中表示出来． 为了解决这一问题，可以采用极坐标系．      

**极坐标参数空间**       
