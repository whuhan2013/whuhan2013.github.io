---
layout: post
title: Opencv之Core组件
date: 2017-1-8
categories: blog
tags: [Opencv]
description: Opencv之core组件
---

了解过之前老版本OpenCV的童鞋们都应该清楚，对于OpenCV1.0时代的基于C语言接口而建的图像存储格式IplImage*，如果在退出前忘记release掉的话，就会照成内存泄露.自从OpenCV踏入2.0时代，用Mat类数据结构作为主打之后，OpenCV变得越发像需要很少编程涵养的Matlab那样，上手超级快。甚至有些函数名称都和matlab一样，比如大家所熟知的imread，imwrite，imshow等函数。        

**mat类型**      
cv::Mat类是用于保存图像以及其他矩阵数据的数据结构。默认情况下，其尺寸为0，我们也可以指定初始尺寸,比如，比如定义一个Mat类对象，就要写cv::Mat pic(320,640,cv::Scalar(100));           
Mat有多种创建方法，此处省略。       

**其他常用数据结构**      

- 二维点：Point2f    
- 三维点: Point3f    
- 点的表示：Point    
- 颜色的表示：Scalar类     
- 尺寸的表示：Size类     
- 矩形的表示：Rect类      
- 颜色空间转换：cvtColor      

#### 基本图形的绘制     
 
**画椭圆**     

```
void DrawEllipse( Mat img, double angle )
{
    int thickness = 2;
    int lineType = 8;

    ellipse( img,
             Point( WINDOW_WIDTH/2, WINDOW_WIDTH/2 ),
             Size( WINDOW_WIDTH/4, WINDOW_WIDTH/16 ),
             angle,
             0,
             360,
             Scalar( 255, 129, 0 ),
             thickness,
             lineType );
}
```

**画圆**        

```
void DrawFilledCircle( Mat img, Point center )
{
    int thickness = -1;
    int lineType = 8;

    circle( img,
            center,
            WINDOW_WIDTH/32,
            Scalar( 0, 0, 255 ),
            thickness,
            lineType );
}
```

**画多边形**     

```
void DrawPolygon( Mat img )
{
    int lineType = 8;

    //创建一些点
    Point rookPoints[1][20];
    rookPoints[0][0]  = Point(    WINDOW_WIDTH/4,   7*WINDOW_WIDTH/8 );
    rookPoints[0][1]  = Point(  3*WINDOW_WIDTH/4,   7*WINDOW_WIDTH/8 );
    rookPoints[0][2]  = Point(  3*WINDOW_WIDTH/4,  13*WINDOW_WIDTH/16 );
    rookPoints[0][3]  = Point( 11*WINDOW_WIDTH/16, 13*WINDOW_WIDTH/16 );
    rookPoints[0][4]  = Point( 19*WINDOW_WIDTH/32,  3*WINDOW_WIDTH/8 );
    rookPoints[0][5]  = Point(  3*WINDOW_WIDTH/4,   3*WINDOW_WIDTH/8 );
    rookPoints[0][6]  = Point(  3*WINDOW_WIDTH/4,     WINDOW_WIDTH/8 );
    rookPoints[0][7]  = Point( 26*WINDOW_WIDTH/40,    WINDOW_WIDTH/8 );
    rookPoints[0][8]  = Point( 26*WINDOW_WIDTH/40,    WINDOW_WIDTH/4 );
    rookPoints[0][9]  = Point( 22*WINDOW_WIDTH/40,    WINDOW_WIDTH/4 );
    rookPoints[0][10] = Point( 22*WINDOW_WIDTH/40,    WINDOW_WIDTH/8 );
    rookPoints[0][11] = Point( 18*WINDOW_WIDTH/40,    WINDOW_WIDTH/8 );
    rookPoints[0][12] = Point( 18*WINDOW_WIDTH/40,    WINDOW_WIDTH/4 );
    rookPoints[0][13] = Point( 14*WINDOW_WIDTH/40,    WINDOW_WIDTH/4 );
    rookPoints[0][14] = Point( 14*WINDOW_WIDTH/40,    WINDOW_WIDTH/8 );
    rookPoints[0][15] = Point(    WINDOW_WIDTH/4,     WINDOW_WIDTH/8 );
    rookPoints[0][16] = Point(    WINDOW_WIDTH/4,   3*WINDOW_WIDTH/8 );
    rookPoints[0][17] = Point( 13*WINDOW_WIDTH/32,  3*WINDOW_WIDTH/8 );
    rookPoints[0][18] = Point(  5*WINDOW_WIDTH/16, 13*WINDOW_WIDTH/16 );
    rookPoints[0][19] = Point(    WINDOW_WIDTH/4,  13*WINDOW_WIDTH/16 );

    const Point* ppt[1] = { rookPoints[0] };
    int npt[] = { 20 };

    fillPoly( img,
              ppt,
              npt,
              1,
              Scalar( 255, 255, 255 ),
              lineType );
}
```

**画线**      

```
void DrawLine( Mat img, Point start, Point end )
{
    int thickness = 2;
    int lineType = 8;
    line( img,
          start,
          end,
          Scalar( 0, 0, 0 ),
          thickness,
          lineType );
}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter2/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter2/p2.png)


**用指针访问像素**      

```
void colorReduce(Mat& inputImage, Mat& outputImage, int div)  
{  
    //参数准备
    outputImage = inputImage.clone();  //拷贝实参到临时变量
    int rowNumber = outputImage.rows;  //行数
    int colNumber = outputImage.cols*outputImage.channels();  //列数 x 通道数=每一行元素的个数

    //双重循环，遍历所有的像素值
    for(int i = 0;i < rowNumber;i++)  //行循环
    {  
        uchar* data = outputImage.ptr<uchar>(i);  //获取第i行的首地址
        for(int j = 0;j < colNumber;j++)   //列循环
        {   
            // ---------【开始处理每个像素】-------------     
            data[j] = data[j]/div*div + div/2;  
            // ----------【处理结束】---------------------
        }  //行处理结束
    }  
}  
```

**用迭代器访问像素**      

```
void colorReduce(Mat& inputImage, Mat& outputImage, int div)  
{  
    //参数准备
    outputImage = inputImage.clone();  //拷贝实参到临时变量
    //获取迭代器
    Mat_<Vec3b>::iterator it = outputImage.begin<Vec3b>();  //初始位置的迭代器
    Mat_<Vec3b>::iterator itend = outputImage.end<Vec3b>();  //终止位置的迭代器

    //存取彩色图像像素
    for(;it != itend;++it)  
    {  
        // ------------------------【开始处理每个像素】--------------------
        (*it)[0] = (*it)[0]/div*div + div/2;  
        (*it)[1] = (*it)[1]/div*div + div/2;  
        (*it)[2] = (*it)[2]/div*div + div/2;  
        // ------------------------【处理结束】----------------------------
    }  
}  
```

**用动态地址计算配合at访问像素**      

```
void colorReduce(Mat& inputImage, Mat& outputImage, int div)  
{  
    //参数准备
    outputImage = inputImage.clone();  //拷贝实参到临时变量
    int rowNumber = outputImage.rows;  //行数
    int colNumber = outputImage.cols;  //列数

    //存取彩色图像像素
    for(int i = 0;i < rowNumber;i++)  
    {  
        for(int j = 0;j < colNumber;j++)  
        {   
            // ------------------------【开始处理每个像素】--------------------
            outputImage.at<Vec3b>(i,j)[0] =  outputImage.at<Vec3b>(i,j)[0]/div*div + div/2;  //蓝色通道
            outputImage.at<Vec3b>(i,j)[1] =  outputImage.at<Vec3b>(i,j)[1]/div*div + div/2;  //绿色通道
            outputImage.at<Vec3b>(i,j)[2] =  outputImage.at<Vec3b>(i,j)[2]/div*div + div/2;  //红是通道
            // -------------------------【处理结束】----------------------------
        }  // 行处理结束     
    }  
}  
```

注意：opencv中的彩色图像不是按rgb顺序排列的，而是bgr。       

#### ROI与图像混合       

定义ROI感兴趣区域有两种方法：      

```
//方法一  
imageROI=image(Rect(500,250,logo.cols,logo.rows));  
//方法二  
imageROI=srcImage3(Range(250,250+logoImage.rows),Range(200,200+logoImage.cols));  
```

**线性混合**     
线性混合操作是一种典型的二元（两个输入）的像素操作，它的理论公式是这样的：      
$$g(x)=(1-a)f_a(x)+af_3(x)$$        
我们通过在范围0到1之间改变alpha值，来对两幅图像（f0（x）和f1（x））或两段视频（同样为（f0（x）和f1（x））产生时间上的画面叠化（cross-dissolve）效果，就像幻灯片放映和电影制作中的那样。即在幻灯片翻页时设置的前后页缓慢过渡叠加效果，以及电影情节过渡时经常出现的画面叠加效果。
实现方面，我们主要运用了OpenCV中addWeighted函数.                  

```
bool  LinearBlending()
{
    //【0】定义一些局部变量
    double alphaValue = 0.5; 
    double betaValue;
    Mat srcImage2, srcImage3, dstImage;

    // 【1】读取图像 ( 两幅图片需为同样的类型和尺寸 )
    srcImage2 = imread("mogu.jpg");
    srcImage3 = imread("rain.jpg");

    if( !srcImage2.data ) { printf("读取srcImage2错误！ \n"); return false; }
    if( !srcImage3.data ) { printf("读取srcImage3错误！ \n"); return false; }

    // 【2】进行图像混合加权操作
    betaValue = ( 1.0 - alphaValue );
    addWeighted( srcImage2, alphaValue, srcImage3, betaValue, 0.0, dstImage);

    // 【3】显示原图窗口
    imshow( "<2>线性混合示例窗口【原图】", srcImage2 );
    imshow( "<3>线性混合示例窗口【效果图】", dstImage );

    return true;

}
```

#### 分离颜色通道，多通道颜色混合        

**通道分离split函数**        

```
    //【2】将一个三通道图像转换成三个单通道图像
    split(srcImage,channels);//分离色彩通道

    //【3】将原图的红色通道引用返回给imageBlueChannel，注意是引用，相当于两者等价，修改其中一个另一个跟着变
    imageRedChannel= channels.at(2);
```   

**通道合并，merge函数**           
merge是split的逆向操作，是把多个通道合并       

```
    //【5】将三个单通道重新合并成一个三通道
    merge(channels,srcImage);
```

#### 图像对比度、亮度值调整      

```
static void ContrastAndBright(int, void *)
{

    // 创建窗口
    namedWindow("【原始图窗口】", 1);

    // 三个for循环，执行运算 g_dstImage(i,j) = a*g_srcImage(i,j) + b
    for( int y = 0; y < g_srcImage.rows; y++ )
    {
        for( int x = 0; x < g_srcImage.cols; x++ )
        {
            for( int c = 0; c < 3; c++ )
            {
                g_dstImage.at<Vec3b>(y,x)[c] = saturate_cast<uchar>( (g_nContrastValue*0.01)*( g_srcImage.at<Vec3b>(y,x)[c] ) + g_nBrightValue );
            }
        }
    }

    // 显示图像
    imshow("【原始图窗口】", g_srcImage);
    imshow("【效果图窗口】", g_dstImage);
}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter2/p3.png)

**离散傅立叶变换**       
dft函数用于对一维或二维浮点数组进行正向或反向傅立叶变换。       



