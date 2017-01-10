---
layout: post
title: Opencv之图像处理
date: 2017-1-10
categories: blog
tags: [Opencv]
description: Opencv之图像处理
---

图像线性滤波实现，主要包括以下三种     

- 方框滤波    
- 平均滤波   
- 高斯滤波      

方框滤波就是没有归一化的均值滤波  

```
void boxFilter(){
    // 载入原图
    Mat image=imread("../img/1.jpg");

    //创建窗口
    namedWindow( "方框滤波【原图】" );
    namedWindow( "方框滤波【效果图】");

    //显示原图
    imshow( "方框滤波【原图】", image );

    //进行方框滤波操作
    Mat out;
    boxFilter( image, out, -1,Size(5, 5));

    //显示效果图
    imshow( "方框滤波【效果图】" ,out );

    waitKey( 0 );
}
```

均值滤波实现    

```
void averageFilter(){
    //【1】载入原始图
    Mat srcImage=imread("../img/1.jpg");

    //【2】显示原始图
    imshow( "均值滤波【原图】", srcImage );

    //【3】进行均值滤波操作
    Mat dstImage;
    blur( srcImage, dstImage, Size(7, 7));

    //【4】显示效果图
    imshow( "均值滤波【效果图】" ,dstImage );

    waitKey( 0 );
}
```

高斯滤波实现     

```
void gaussinFilter(){
    // 载入原图
    Mat image=imread("../img/1.jpg");

    //创建窗口
    namedWindow( "高斯滤波【原图】" );
    namedWindow( "高斯滤波【效果图】");

    //显示原图
    imshow( "高斯滤波【原图】", image );

    //进行高斯滤波操作
    Mat out;
    GaussianBlur( image, out, Size( 5, 5 ), 0, 0 );

    //显示效果图
    imshow( "高斯滤波【效果图】" ,out );

    waitKey( 0 );
}
```

**非线性滤波实现**      
主要包括均值滤波与双边滤波。          
中值滤波（Median filter）是一种典型的非线性滤波技术，基本思想是用像素点邻域灰度值的中值来代替该像素点的灰度值，该方法在去除脉冲噪声、椒盐噪声的同时又能保留图像边缘细节，．               
双边滤波（Bilateral filter）是一种非线性的滤波方法，是结合图像的空间邻近度和像素值相似度的一种折衷处理，同时考虑空域信息和灰度相似性，达到保边去噪的目的。具有简单、非迭代、局部的特点。          
双边滤波器的好处是可以做边缘保存（edge preserving），一般过去用的维纳滤波或者高斯滤波去降噪，都会较明显地模糊边缘，对于高频细节的保护效果并不明显。双边滤波器顾名思义比高斯滤波多了一个高斯方差sigma－d，它是基于空间分布的高斯滤波函数，所以在边缘附近，离的较远的像素不会太多影响到边缘上的像素值，这样就保证了边缘附近像素值的保存。但是由于保存了过多的高频信息，对于彩色图像里的高频噪声，双边滤波器不能够干净的滤掉，只能够对于低频信息进行较好的滤波。

中值滤波 

```
void centerFilter(){
    // 载入原图
    Mat image=imread("../img/1.jpg");

    //创建窗口
    namedWindow( "中值滤波【原图】" );
    namedWindow( "中值滤波【效果图】");

    //显示原图
    imshow( "中值滤波【原图】", image );

    //进行中值滤波操作
    Mat out;
    medianBlur ( image, out, 7);

    //显示效果图
    imshow( "中值滤波【效果图】" ,out );

    waitKey( 0 );
}
```

双边滤波      

```
void doubleFilter(){
    // 载入原图
    Mat image=imread("../img/1.jpg");

    //创建窗口
    namedWindow( "双边滤波【原图】" );
    namedWindow( "双边滤波【效果图】");

    //显示原图
    imshow( "双边滤波【原图】", image );

    //进行双边滤波操作
    Mat out;
    bilateralFilter ( image, out, 25, 25*2, 25/2 );

    //显示效果图
    imshow( "双边滤波【效果图】" ,out );

    waitKey( 0 );
}
```

#### 形态学滤波       
主要包括腐蚀与膨胀        
