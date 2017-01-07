---
layout: post
title: Opencv学习入门
date: 2017-1-6
categories: blog
tags: [Opencv]
description: Opencv学习入门
---

**Opencv基本架构**       

(1)【calib3d】——其实就是就是Calibration（校准）加3D这两个词的组合缩写。这个模块主要是相机校准和三维重建相关的内容。基本的多视角几何算法，单个立体摄像头标定，物体姿态估计，立体相似性算法，3D信息的重建等等。
 
(2)【contrib】——也就是Contributed/Experimental Stuf的缩写， 该模块包含了一些最近添加的不太稳定的可选功能，不用去多管。2.4.8里的这个模块有新型人脸识别，立体匹配，人工视网膜模型等技术。
 
(3)【core】——核心功能模块，包含如下内容：         
OpenCV基本数据结构      
动态数据结构      
绘图函数          
数组操作相关函数         
辅助功能与系统函数和宏       
与OpenGL的互操作      


(4)【imgproc】——Image和Processing这两个单词的缩写组合。图像处理模块，这个模块包含了如下内容：          
线性和非线性的图像滤波            
图像的几何变换                 
其它（Miscellaneous）图像转换       
直方图相关       
结构分析和形状描述      
运动分析和对象跟踪      
特征检测       
目标检测等内容      

 
(5)【features2d】 ——也就是Features2D， 2D功能框架 ，包含如下内容：      
特征检测和描述                        
特征检测器（Feature Detectors）通用接口       
描述符提取器（Descriptor Extractors）通用接口      
描述符匹配器（Descriptor Matchers）通用接口       
通用描述符（Generic Descriptor）匹配器通用接口        
关键点绘制函数和匹配功能绘制函数       

(6)【flann】—— Fast Library for Approximate Nearest Neighbors，高维的近似近邻快速搜索算法库，包含两个部分：
快速近似最近邻搜索           
聚类        

(7)【gpu】——运用GPU加速的计算机视觉模块
 
(8)【highgui】——也就是high gui，高层GUI图形用户界面，包含媒体的I / O输入输出，视频捕捉、图像和视频的编码解码、图形交互界面的接口等内容
 
(9)【legacy】——一些已经废弃的代码库，保留下来作为向下兼容，包含如下相关的内容：       
运动分析      
期望最大化       
直方图            
平面细分（C API）                                 
特征检测和描述（Feature Detection and Description）     
描述符提取器（Descriptor Extractors）的通用接口       
通用描述符（Generic Descriptor Matchers）的常用接口      
匹配器     
 
(10)【ml】——Machine Learning，机器学习模块， 基本上是统计模型和分类算法，包含如下内容：     
统计模型 （Statistical Models）             
一般贝叶斯分类器 （Normal Bayes Classifier）      
K-近邻 （K-NearestNeighbors）        
支持向量机 （Support Vector Machines）       
决策树 （Decision Trees）      
提升（Boosting）                  
梯度提高树（Gradient Boosted Trees）        
随机树 （Random Trees）                
超随机树 （Extremely randomized trees）     
期望最大化 （Expectation Maximization）      
神经网络 （Neural Networks）       
MLData        

(9)【nonfree】，也就是一些具有专利的算法模块 ，包含特征检测和GPU相关的内容。最好不要商用，可能会被告哦。
 
(10)【objdetect】——目标检测模块，包含Cascade Classification（级联分类）和Latent SVM这两个部分。
 
(11)【ocl】——即OpenCL-accelerated Computer Vision，运用OpenCL加速的计算机视觉组件模块
 
(12)【photo】——也就是Computational Photography，包含图像修复和图像去噪两部分
 
(13)【stitching】——images stitching，图像拼接模块，包含如下部分：            
拼接流水线       
特点寻找和匹配图像         
估计旋转         
自动校准      
图片歪斜      
接缝估测      
曝光补偿       
图片混合    

(14)【superres】——SuperResolution，超分辨率技术的相关功能模块
 
(15)【ts】——opencv测试相关代码，不用去管他
 
(16)【video】——视频分析组件，该模块包括运动估计，背景分离，对象跟踪等视频处理相关内容。
 
(17)【Videostab】——Video stabilization，视频稳定相关的组件，官方文档中没有多作介绍，不管它了。

### opencv图像处理快速上手      

**显示图片**     

```
void quickStart(){
    // 【1】读入一张图片
    Mat img=imread("../img/1.jpg");
    // 【2】在窗口中显示载入的图片
    imshow("【载入的图片】",img);
    // 【3】等待6000 ms后窗口自动关闭
    waitKey(6000);
}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter1/p1.png)  

**图像腐蚀**      

```
void myerode(){
    //载入原图
    Mat srcImage = imread("../img/2.jpg");
    //显示原图
    imshow("【原图】腐蚀操作", srcImage);
    //进行腐蚀操作
    Mat element = getStructuringElement(MORPH_RECT, Size(15, 15));
    Mat dstImage;
    erode(srcImage, dstImage, element);
    //显示效果图
    imshow("【效果图】腐蚀操作", dstImage);
    waitKey(0);

}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter1/p2.png) 

**图像模糊**      

```
void myblur(){
    //【1】载入原始图
    Mat srcImage=imread("../img/3.jpg");

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
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter1/p3.png) 

**canny算子边缘检测**     

```
void mycanny(){
    //【0】载入原始图
    Mat srcImage = imread("../img/4.jpg");  //工程目录下应该有一张名为1.jpg的素材图
    imshow("【原始图】Canny边缘检测", srcImage);   //显示原始图
    

    Mat dstImage,edge,grayImage;  //参数定义

    //【1】创建与src同类型和大小的矩阵(dst)
    dstImage.create( srcImage.size(), srcImage.type() );

    //【2】将原图像转换为灰度图像
    //此句代码的OpenCV2版为：
    //cvtColor( srcImage, grayImage, CV_BGR2GRAY );
    //此句代码的OpenCV3版为：
    cvtColor( srcImage, grayImage, COLOR_BGR2GRAY );

    //【3】先用使用 3x3内核来降噪
    blur( grayImage, edge, Size(3,3) );

    //【4】运行Canny算子
    Canny( edge, edge, 3, 9,3 );

    //【5】显示效果图
    imshow("【效果图】Canny边缘检测", edge);

    waitKey(0);
}
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter1/p4.png) 


**调用摄像头与canny边缘检测结合**       

```
void mycamera(){
    //【1】从摄像头读入视频
    VideoCapture capture(0);
    Mat edges;

    //【2】循环显示每一帧
    while(1)
    {
        Mat frame;  //定义一个Mat变量，用于存储每一帧的图像
        capture>>frame;  //读取当前帧
        cvtColor(frame,edges,CV_BGR2GRAY);
        blur(edges,edges,Size(7,7));
        Canny(edges,edges,0,30,3);
        imshow("读取视频",edges);  //显示当前帧
        waitKey(30);  //延时30ms
    }

}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter1/p5.png) 




