---
layout: post
title: 车牌定位实现
date: 2017-1-13
categories: blog
tags: [Opencv]
description: 车牌定位
---

**车牌定位实现过程**        
plateLocate的总体识别思路是：如果我们的车牌没有大的旋转或变形，那么其中必然包括很多垂直边缘（这些垂直边缘往往缘由车牌中的字符），如果能够找到一个包含很多垂直边缘的矩形块，那么有很大的可能性它就是车牌。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p4.jpg)

**主要步骤**      
1、高斯模糊     
2、经过高斯模糊后的图片。经过这步处理，可以看出图像变的模糊了。这步的作用是为接下来的Sobel算子去除干扰的噪声。    
3、将图像进行灰度化。这个步骤是一个分水岭，意味着后面的所有操作都不能基于色彩信息了。       
4、对图像进行Sobel运算，得到的是图像的一阶水平方向导数。这步过后，车牌被明显的区分出来。        
5、对图像进行二值化。将灰度图像（每个像素点有256个取值可能）转化为二值图像（每个像素点仅有1和0两个取值可能）。     
6、使用闭操作。对图像进行闭操作以后，可以看到车牌区域被连接成一个矩形装的区域。      
7、求轮廓。求出图中所有的轮廓。这个算法会把全图的轮廓都计算出来，因此要进行筛选。      
8、筛选。对轮廓求最小外接矩形，然后验证，不满足条件的淘汰。       
9、角度判断与旋转。把倾斜角度大于阈值（如正负30度）的矩形舍弃。余下的矩形进行微小的旋转，使其水平。        
10、统一尺寸。上步得到的图块尺寸是不一样的。为了进入机器学习模型，需要统一尺寸。统一尺寸的标准宽度是136，长度是36。这个标准是对千个测试车牌平均后得出的通用值。         
这些“车牌”有两个作用：一、积累下来作为支持向量机（SVM）模型的训练集，以此训练出一个车牌判断模型；二、在实际的车牌检测过程中，将这些候选“车牌”交由训练好的车牌判断模型进行判断。如果车牌判断模型认为这是车牌的话就进入下一步即字符识别过程，如果不是，则舍弃。        

**车牌定位头文件**     

```
class CPlateLocate 
{
public:
    CPlateLocate();

    //! 车牌定位
    int plateLocate(Mat, vector<Mat>& );

    //! 车牌的尺寸验证
    bool verifySizes(RotatedRect mr);

    //! 结果车牌显示
    Mat showResultMat(Mat src, Size rect_size, Point2f center);

    //! 设置与读取变量
    //...

protected:
    //! 高斯模糊所用变量
    int m_GaussianBlurSize;

    //! 连接操作所用变量
    int m_MorphSizeWidth;
    int m_MorphSizeHeight;

    //! verifySize所用变量
    float m_error;
    float m_aspect;
    int m_verifyMin;
    int m_verifyMax;

    //! 角度判断所用变量
    int m_angle;

    //! 是否开启调试模式，0关闭，非0开启
    int m_debug;
};
```       

原图像    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p5.jpg)

**高斯模糊**      
GaussianBlur( src, out, Size( getM_GaussianBlurSize(), getM_GaussianBlurSize() ), 0, 0 );     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p6.jpg)

