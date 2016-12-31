---
layout: post
title: 形态学图像处理(二)
date: 2016-12-31
categories: blog
tags: [图像处理]
description: 形态学图像处理
---


**本文主要包括以下内容**    

- 二值形态学的经典应用， 细化和像素化， 以及凸壳
- 灰度图像的形态学运算， 包括灰度腐蚀、灰度膨胀、灰度开和灰度闭
- 本章的典型案例分析
  + 在人脸局部图像中定位嘴的中心
  + 显微镜下图像的细菌计数
  + 利用顶帽变换（top-hat）技术解决光照不均问题  

#### 细化算法          
“骨架”是指一副图像的骨髓部分，它描述物体的几何形状和拓扑结构，是重要的图像描绘子之一，计算骨架的过程一般称为“细化”或“骨架化”，在包括文字识别、工业零件形状识别以及印刷电路板自动检测在内的很多应用中，细化过程都发挥着关键作用。通常，对我们感兴趣的目标物体进行细化有助于突出目标的形状特点和拓扑结构并且减少冗余的信息量。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p1.png)   
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p2.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p3.png) 

#### 像素化算法          
细化适用于和物体拓扑结构或形状有关的应用， 如前述的手写字符识别。 但有时我们关心的是目标对象是否存在、 它们的位置关系， 或者个数， 这时在预处理中加入像素化步骤会给后续的图像分析带来极大的方便             

**理论基础**      
像素化操作首先需找到二值图像中所有的连通区域， 然后用这些区域的质心作为这些连通区域的代表， 即将1个连通区域像素化为位于区域质心位置的1个像素。  
有时还可以进一步引入一个低阀值lowerThres和一个高阙值highThres来指出图像中我们感兴趣的对象连通数(连通分量中的像素数目)的大致范围， 从而只像素化图像中大小介于lowerThres和upperThres之间的连通区域， 而连通数低于lowerThres或高于upperThres的对象都将被滤除， 这就相当于使用算法的同时具有了过滤噪声的能力。如图8.29 所示。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p4.png) 

#### 凸壳     
如果连接物体A内任意两点的直线段都在淫的内部，则称d是凸的。任意物体A的凸亮H是包含A的最小凸物体。       
我们总是希望像素化算法能够找到物体的质心来代表读物体，但在实际中，可能由于光照不均等原因导致图像在二值化后，物体本身形状发生缺损，像素化算法就无法找到物体真正的质心。此时可适当地进行凸壳处理，弥补凹损，算法会找到包含原始形状的最小凸多边形，如下图8.30所示．           
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p5.png)
为确保在上述生长过程中凸壳不会大幅超出凸性所需的最小尺寸， 可以限制其生长以便凸壳不会超出初始时包含物体A的最小矩形．      

**bwmorph函数**          
本章的很多形态学操作都可由IPT函数bwmorph实现， 该函数的调用语法为：               
Iout = bwmorph(I,operation,n);             
operation是一个指定操作类型的字符串， 常用的合法取位如在8.2 所示          
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p6.png)  

### 灰度图像中的基本形态学运算      
本节就们把二值图像的形态学处理扩展到灰度图像的基本操作， 包括灰度膨胀、灰皮腐蚀、灰度开和灰皮闭。此外， 8.4.4 小节还将介绍一个灰度形态学的经典应用一一顶帽变换(top-hat）， 用以解决图像的光照不均问题．        

#### 灰度膨胀及其实现     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p7.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p8.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter82/p9.png) 

**matlab实现**      
