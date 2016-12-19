---
layout: post
title: 灰度变换与空间滤波
date: 2016-12-19
categories: blog
tags: [图像处理]
description: 灰度变换与空间滤波
---

术语“空间域”指的是图像平面本身，这类方法是以对图像像素直接处理为基础的。在本章中，我们着重讨论两种重要的空间域处理方法z亮度（或灰度）变换与空间滤波。后一种方法有时涉及邻域处理或空间卷积。    

**背景知识**      
前面已经指出，空间域技术直接对图像的像素进行操作。本章中讨论的空间域处理由F列表达式表示：      
$$g (x, y) = T [f(x, y)] $$      

其中，f(x,y）为输入图像，g(x,y）为输出（处理后的）图像，T是对图像f的算子，作用于点。(x,y)定义的邻域。此外，T还可以对一组图像进行处理，例如为了降低噪声而叠加K幅图像。  

为了定义点（x,y）的空间邻域，主要方法是利用一块中心位于（x,y）的正方形或矩形区域，如图二l所示。此区域的中心由起始点开始逐个像素移动，比如从左上角，在移动的同时包含不同的邻域。算子T作用于每个位置.(x,y），从而得到相应位置的输出图像g.只有中心点在(x,y)处的邻域内的像素被用来计算(x,y）处g的值。    

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p2.png)  

### 灰度变换函数       
变换 T 的最简单形式是如图2-1 中邻域大小为 1 ×1（单个像素）的情况。在此 情况下，在(x,y)处，g的值仅由f在这 点处的灰度决定， T也就成为亮度或灰度变换函数。 当处理单色（也就是灰度）图像时，这两个术语是可以相互换 用的。     
当处理彩色图像时， 亮度用来表示在特定色彩空间里的彩色图像成分， 正像在第6章中描述的那样。
由于输出值仅取决于点的灰度值， 而不是取决于点的邻域， 因此 灰度变换函数通常写成如F简单形式z    
$$s = T(r) $$       
其中，r表示图像f中的灰度，s表示图像g中的灰度。 两者在图像中处于相同的坐帧x,y）处。     

**imadjust和stretchlim函数**    

imadjust函数是针对灰度图像进行灰度变换的基本图像处理工具箱函数， 一般的语法格
式如下：      
g = imadjust(f, [low_in high_in], [low_out high_out],gamma)  

正如图2-2中展示的那样，此函数将f的灰度值映像到q中的新值， 也就是将low in 与
high_in之间的值映射到low_out 与high_out 之间的值。low_in以下与high_in以上
的值可以被截去。也就是将low_in以下的值映射为low_out：将high in以上的值映射为
high out. 输入图像应属于uint8、uintl6或double类。输出图像应和输入图像属于同一
类。对于函数imadjust来说， 所有输入中除了图像f和gamma， 不论f属于什么类， 都将输入
值限定在0和1之间。例如， 如果f属于uint8类， imadjust函数将乘以255来决定应用中的
实际值。利用空矩阵（［］）得到［low_in high_in］或［low_out high_out ］， 将导致结果都默认
为［0 I ］。如果high_out 小于low_out ， 输出灰度将反转。

参数gamma指明了由f映射生成图像q时曲线的形状。如果gamma的值小于1， 映射被
加权至较高（较亮）的输出值， 如图2-2(a）所示。如果gamma的值大于1， 映射加权至较低（较暗）
的输出值。如果省略函数参量， gamma默认为I（线性映射）。


![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter2/p3.png)  

stretchlim函数的较通用语法如下：      
Low H工gh = stretchlim(f, tol)       

其中， tol是两元素向量［low_frac high frac］， 指定了图像低和高像素值 饱和度的 百分比。
如果tol 是标量， 那么low_frac = tol， 并且high_frac = l - low frac；     

**饱和度**等于低像素值和高像素值的百分比． 如果在参数中忽略tol， 那么饱和度水平为2%, tol 的默认值为［0.01 0.99］。 如果选择tol =O， 那么Low_High = [min(f(:)) max(f（：））］。


