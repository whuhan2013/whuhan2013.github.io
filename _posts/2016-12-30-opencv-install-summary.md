---
layout: post
title: Opencv安装总结
date: 2016-12-30
categories: blog
tags: [Opencv]
description: Opencv安装总结
---

**mac系统**     

1.下载Homebrew  在Terminal中输入：                           
ruby -e "$(curl -fsSkL raw.github.com/mxcl/homebrew/go)"             
2.安装cmake       在Terminal中输入：           
brew install cmake     
3.开始安装opneCV          
brew install homebrew/science/opencv3    

**问题**      
1.权限问题    

```
sudo chown -R $(whoami) /usr/local/Cellar
sudo chown -R $(whoami) /Users/schuers/Library
```

2.报错      

```
dyld: Library not loaded: /usr/local/opt/webp/lib/libwebp.6.dylib
  Referenced from: /usr/local/opt/opencv3/lib/libopencv_imgcodecs.3.1.dylib
   Reason: image not found
[1]    17647 trace trap  bin/DisplayImage 011.jpg
```

解决:参见[http://answers.opencv.org/question/98455/mac-launch-issue/](http://answers.opencv.org/question/98455/mac-launch-issue/)


**mac下qt配置opencv程序**      
参见：这个应该是比较权威的配置描述了，省了许多弯路：[https://www.learnopencv.com/configuring-qt-for-opencv-on-osx/](https://www.learnopencv.com/configuring-qt-for-opencv-on-osx/)     

```
#include "opencv2/opencv.hpp"

int main()
{
    cv::Mat img = cv::imread("/Users/schuser/beauty.jpg");
    cv::imshow("Image", img);
    cv::waitKey(0);

    return 0;
}
```

**clion配置opencv**       

在cmakelists.txt中添加     

```
find_package( OpenCV REQUIRED )
target_link_libraries( 工程名 ${OpenCV_LIBS} )
```

配置cmake环境变量:OpenCV_DIR=/usr/local/Cellar/opencv3/3.1.0_4/share/OpenCV/                  
修改/usr/local/Cellar/opencv3/3.1.0_4/share/OpenCV/路径下OpenCVConfig.cmake，在第一行添加set(OpenCV_FOUND 1)      
**参见：**[CLion中配置OpenCV环境问题](http://blog.csdn.net/shenck1992/article/details/49757693)

实例     

```
#include <cv.hpp>

int main()
{
    IplImage* img = cvLoadImage("/Users/schuser/beauty.jpg");
    int x = 150, y = 80;
    int width = 150, height = 150;
    int add =180;
    cvSetImageROI(img, cvRect(x, y, width, height));
    cvAddS(img, cvScalar(add), img);
    cvResetImageROI(img);
    cvShowImage("Image", img);
    cvWaitKey(0);
    return 0;
}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter10b/p4.png)


**cmakeList.txt添加静态库**        
需要添加头文件与lib即可。       

```
include_directories(include)
link_directories(lib)
target_link_libraries(CarVisual easypr thirdparty ${OpenCV_LIBS} )
```

**CImg库的编译**      
1.下载XQuartz获得x11。     
2. 进入example目录下编译:g++ -o test CImg_demo.cpp -lX11 -lpthread -L/usr/X11/lib -I/usr/X11/include

**opencv3使用vector报错**     
解决方法：添加using namespace std即可。      

OpenCV3.0.0API文档中英文对照

```
OpenCV API Reference
Introduction
API Concepts
core. The Core Functionality
Basic Structures
Command Line Parser  分析器
Basic C Structures and Operations
Dynamic Structures
Operations on Arrays
XML/YAML Persistence
XML/YAML Persistence (C API)
Clustering  集群
Utility and System Functions and Macros 辅助 系统 函数 宏
OpenGL interoperability  互操作性
Intel® IPP Asynchronous C/C++ Converters 异步C++转换器
Optimization Algorithms  
imgproc. Image Processing
Image Filtering
Geometric Image Transformations
Miscellaneous Image Transformations  多方面图像转换
Drawing Functions  
ColorMaps in OpenCV  色图
Description  
Histograms  直方图
Structural Analysis and Shape Descriptors
Motion Analysis and Object Tracking
Feature Detection  特征
Object Detection  目标
imgcodecs. Image file reading and writing
Reading and Writing Images
videoio. Media I/O
Reading and Writing Video
highgui. High-level GUI and Media I/O
User Interface  用户接口
Qt New Functions  
video. Video Analysis
Motion Analysis and Object Tracking
calib3d. Camera Calibration and 3D Reconstruction  摄像机标定  3D重构
Camera Calibration and 3D Reconstruction  
features2d. 2D Features Framework  
Feature Detection and Description
Common Interfaces of Feature Detectors 特征检测器
Common Interfaces of Descriptor Extractors  描述符提取器
Common Interfaces of Descriptor Matchers  描述匹配器
Drawing Function of Keypoints and Matches  绘制关键点和匹配
Object Categorization  对象分类
objdetect. Object Detection
Cascade Classification
ml. Machine Learning
Statistical Models  统计模型
Normal Bayes Classifier 普通贝叶斯分类器
K-Nearest Neighbors  K-近邻
Support Vector Machines
Decision Trees  决策树
Boosting
Random Trees  
Expectation Maximization    期望最大化
Neural Networks  神经网络
Logistic Regression  Logistic回归
Training Data
flann. Clustering and Search in Multi-Dimensional Spaces   flann  集群和搜索多维空间
Fast Approximate Nearest Neighbor Search 快速近似最近邻搜索
Clustering
photo. Computational Photography  计算摄影
Inpainting  修补
Denoising  去噪
HDR imaging HDR成像
Decolorization 脱色
Seamless Cloning 无缝克隆
Non-Photorealistic Rendering  非真实感绘制
stitching. Images stitching  拼接
Stitching Pipeline  拼接管道
References
High Level Functionality
Camera 
Features Finding and Images Matching 特征寻找 图像匹配
Rotation Estimation  旋转估计
Autocalibration  自动校正
Images Warping  图像整形
Seam Estimation  缝估计
Exposure Compensation  曝光补偿
Image Blenders  图像混合
cuda. CUDA-accelerated Computer Vision CUDA加速
CUDA Module Introduction 
Initalization and Information
Data Structures
Object Detection
Camera Calibration and 3D Reconstruction
cudaarithm. CUDA-accelerated Operations on Matrices  CUDA在矩阵上的加速
Core Operations on Matrices
Per-element Operations  每个元素
Matrix Reductions 
Arithm Operations on Matrices 计算
cudabgsegm. CUDA-accelerated Background Segmentation CUDA加速背景分割
Background Segmentation  
cudacodec. CUDA-accelerated Video Encoding/Decoding   CUDA加速视频编码/解码
Video Decoding
Video Encoding
cudafeatures2d. CUDA-accelerated Feature Detection and Description  CUDA加速的特征检测和描述
Feature Detection and Description
cudafilters. CUDA-accelerated Image Filtering  CUDA加速图像滤波
Image Filtering
cudaimgproc. CUDA-accelerated Image Processing   CUDA加速图像处理
Color space processing  色彩空间处理
Histogram Calculation  直方图计算
Hough Transform  Hough变换
Feature Detection  特征检测
Image Processing  图像处理
cudaoptflow. CUDA-accelerated Optical Flow  CUDA光流加速
Optical Flow
cudastereo. CUDA-accelerated Stereo Correspondence CUDA立体通信加速
Stereo Correspondence
cudawarping. CUDA-accelerated Image Warping  CUDA图像变形加速
Image Warping
shape. Shape Distance and Matching  形状距离和匹配
Shape Distance and Common Interfaces
Shape Transformers and Interfaces
Cost Matrix for Histograms Common Interface  价值矩阵的直方图接口
EMD-L1 
superres. Super Resolution 超分辨率
Super Resolution
videostab. Video Stabilization  视频稳定
Introduction
Global Motion Estimation  全局运动估计
Fast Marching Method  快速匹配
viz. 3D Visualizer  3D可视化
Viz 
Widget
bioinspired. Biologically inspired vision models and derivated tools 仿生视觉模型推导和工具
Human retina documentation  视网膜
cvv. GUI for Interactive Visual Debugging of Computer Vision Programs
CVV API Documentation
CVV GUI Documentation
datasets. Framework for working with different datasets 不同数据集
Action Recognition 动作识别
Face Recognition 人脸
Gesture Recognition 手势
Human Pose Estimation 姿势
Image Registration  图像配准
Image Segmentation  图像分割
Multiview Stereo Matching 多视点的立体匹配
Object Recognition 物体识别
Pedestrian Detection 行人
SLAM
Text Recognition  文本
face. Face Recognition  
FaceRecognizer Documentation
Binary descriptors for lines extracted from an image  从图像中提取线做二进制描述
Introduction
A class to represent a line: KeyLine
Computation of binary descriptors
References
Summary
optflow. Optical Flow Algorithms 光流算法
Dense Optical Flow  密集
Motion Templates 
Optical flow input / output  光流输入​​/输出
reg. Image Registration  图像配准
Registration
rgbd. RGB-Depth Processing RGB深加工
Saliency API 卓越的API
Saliency classes:
surface_matching. Surface Matching  表面匹配
Introduction to Surface Matching
Surface Matching Algorithm Through 3D Features  3D特征的表面匹配
Rough Computation of Object Pose Given PPF 基于PPF的物体姿态粗计算
Pose Registration via ICP 通过ICP的姿势配准
Results
A Complete Sample
References
text. Scene Text Detection and Recognition 场景文字检测与识别
Scene Text Detection 
Scene Text Recognition
tracking. Tracking API  追踪API
Long-term optical tracking API  长期的光绪追踪API
UML design:
Tracker classes:  跟踪器类
xfeatures2d. Extra 2D Features Framework   额外的2D功能框架
Experimental 2D Features Algorithms 
Non-free 2D Features Algorithms
ximgproc. Extended Image Processing  扩展的图像处理
Structured forests for fast edge detection  用于快速边缘检测的结构森林
Domain Transform filter  域变换过滤器
Guided Filter  引导过滤器
Adaptive Manifold Filter   多种自适应滤波器
Joint Bilateral Filter   联合双边滤波器
References
Superpixels
xobjdetect. Extended object detection.  扩展目标检测
Integral Channel Features Detector  积分通道检测功能
xphoto. Additional photo processing algorithms  其他照片处理算法
Color balance  色彩平衡
Denoising   去噪
Inpainting   修补
```


