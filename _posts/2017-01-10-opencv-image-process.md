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

