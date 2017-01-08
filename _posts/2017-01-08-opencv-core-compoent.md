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


![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter2/p4.png)
dft函数用于对一维或二维浮点数组进行正向或反向傅立叶变换。      
getOptimalDFTSize返回dft最优尺寸      
copyMakeBorder扩充图片边界      
magnitude计算二维矢量的幅值     
normalize矩阵归一化     

**主要步骤如下**     
【1】以灰度模式读取原始图像并显示      
2】将输入图像延扩到最佳的尺寸，边界用0补充       
3】为傅立叶变换的结果(实部和虚部)分配存储空间       
4】进行就地离散傅里叶变换       
【5】将复数转换为幅值，即=> log(1 + sqrt(Re(DFT(I))^2 + Im(DFT(I))^2))       
6】进行对数尺度(logarithmic scale)缩放       
【7】剪切和重分布幅度图象限          
【8】归一化，用0到1之间的浮点值将矩阵变换为可视的图像格式       
【9】显示效果图     



```
int main( )
{

    //【1】以灰度模式读取原始图像并显示
    Mat srcImage = imread("1.jpg", 0);
    if(!srcImage.data ) { printf("读取图片错误，请确定目录下是否有imread函数指定图片存在~！ \n"); return false; } 
    imshow("原始图像" , srcImage);   

    ShowHelpText();

    //【2】将输入图像延扩到最佳的尺寸，边界用0补充
    int m = getOptimalDFTSize( srcImage.rows );
    int n = getOptimalDFTSize( srcImage.cols ); 
    //将添加的像素初始化为0.
    Mat padded;  
    copyMakeBorder(srcImage, padded, 0, m - srcImage.rows, 0, n - srcImage.cols, BORDER_CONSTANT, Scalar::all(0));

    //【3】为傅立叶变换的结果(实部和虚部)分配存储空间。
    //将planes数组组合合并成一个多通道的数组complexI
    Mat planes[] = {Mat_<float>(padded), Mat::zeros(padded.size(), CV_32F)};
    Mat complexI;
    merge(planes, 2, complexI);         

    //【4】进行就地离散傅里叶变换
    dft(complexI, complexI);           

    //【5】将复数转换为幅值，即=> log(1 + sqrt(Re(DFT(I))^2 + Im(DFT(I))^2))
    split(complexI, planes); // 将多通道数组complexI分离成几个单通道数组，planes[0] = Re(DFT(I), planes[1] = Im(DFT(I))
    magnitude(planes[0], planes[1], planes[0]);// planes[0] = magnitude  
    Mat magnitudeImage = planes[0];

    //【6】进行对数尺度(logarithmic scale)缩放
    magnitudeImage += Scalar::all(1);
    log(magnitudeImage, magnitudeImage);//求自然对数

    //【7】剪切和重分布幅度图象限
    //若有奇数行或奇数列，进行频谱裁剪      
    magnitudeImage = magnitudeImage(Rect(0, 0, magnitudeImage.cols & -2, magnitudeImage.rows & -2));
    //重新排列傅立叶图像中的象限，使得原点位于图像中心  
    int cx = magnitudeImage.cols/2;
    int cy = magnitudeImage.rows/2;
    Mat q0(magnitudeImage, Rect(0, 0, cx, cy));   // ROI区域的左上
    Mat q1(magnitudeImage, Rect(cx, 0, cx, cy));  // ROI区域的右上
    Mat q2(magnitudeImage, Rect(0, cy, cx, cy));  // ROI区域的左下
    Mat q3(magnitudeImage, Rect(cx, cy, cx, cy)); // ROI区域的右下
    //交换象限（左上与右下进行交换）
    Mat tmp;                           
    q0.copyTo(tmp);
    q3.copyTo(q0);
    tmp.copyTo(q3);
    //交换象限（右上与左下进行交换）
    q1.copyTo(tmp);                 
    q2.copyTo(q1);
    tmp.copyTo(q2);

    //【8】归一化，用0到1之间的浮点值将矩阵变换为可视的图像格式
    //此句代码的OpenCV2版为：
    //normalize(magnitudeImage, magnitudeImage, 0, 1, CV_MINMAX); 
    //此句代码的OpenCV3版为:
    normalize(magnitudeImage, magnitudeImage, 0, 1, NORM_MINMAX); 

    //【9】显示效果图
    imshow("频谱幅值", magnitudeImage);    
    waitKey();

    return 0;
}
``` 

傅立叶变换结果          
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/opencv/chapter2/p5.png)

#### 输入输出xml与yaml文件      
一般步骤如下：     
1.实例化FileStorage类对象，完成初始化。     
2.使用流操作符《写入，》读取。       
3.析构FileStorage对象，同时关闭文件。        

**XML和YAML文件的写入**      

```
int main( )  
{  
    //改变console字体颜色
    system("color 5F"); 

    ShowHelpText();

    //初始化
    FileStorage fs("test.yaml", FileStorage::WRITE);  

    //开始文件写入
    fs << "frameCount" << 5;  
    time_t rawtime; time(&rawtime);  
    fs << "calibrationDate" << asctime(localtime(&rawtime));  
    Mat cameraMatrix = (Mat_<double>(3,3) << 1000, 0, 320, 0, 1000, 240, 0, 0, 1);  
    Mat distCoeffs = (Mat_<double>(5,1) << 0.1, 0.01, -0.001, 0, 0);  
    fs << "cameraMatrix" << cameraMatrix << "distCoeffs" << distCoeffs;  
    fs << "features" << "[";  
    for( int i = 0; i < 3; i++ )  
    {  
        int x = rand() % 640;  
        int y = rand() % 480;  
        uchar lbp = rand() % 256;  

        fs << "{:" << "x" << x << "y" << y << "lbp" << "[:";  
        for( int j = 0; j < 8; j++ )  
            fs << ((lbp >> j) & 1);  
        fs << "]" << "}";  
    }  
    fs << "]";  
    fs.release();  

    printf("\n文件读写完毕，请在工程目录下查看生成的文件~");
    getchar();

    return 0;  
}  
```

**XML和YAML文件的读取**      

```
int main( )  
{  
    //改变console字体颜色
    system("color 6F"); 

    ShowHelpText();

    //初始化
    FileStorage fs2("test.yaml", FileStorage::READ);  

    // 第一种方法，对FileNode操作
    int frameCount = (int)fs2["frameCount"];  

    std::string date;  
    // 第二种方法，使用FileNode运算符> > 
    fs2["calibrationDate"] >> date;  

    Mat cameraMatrix2, distCoeffs2;  
    fs2["cameraMatrix"] >> cameraMatrix2;  
    fs2["distCoeffs"] >> distCoeffs2;  

    cout << "frameCount: " << frameCount << endl  
        << "calibration date: " << date << endl  
        << "camera matrix: " << cameraMatrix2 << endl  
        << "distortion coeffs: " << distCoeffs2 << endl;  

    FileNode features = fs2["features"];  
    FileNodeIterator it = features.begin(), it_end = features.end();  
    int idx = 0;  
    std::vector<uchar> lbpval;  

    //使用FileNodeIterator遍历序列
    for( ; it != it_end; ++it, idx++ )  
    {  
        cout << "feature #" << idx << ": ";  
        cout << "x=" << (int)(*it)["x"] << ", y=" << (int)(*it)["y"] << ", lbp: (";  
        // 我们也可以使用使用filenode > > std::vector操作符很容易的读数值阵列
        (*it)["lbp"] >> lbpval;  
        for( int i = 0; i < (int)lbpval.size(); i++ )  
            cout << " " << (int)lbpval[i];  
        cout << ")" << endl;  
    }  
    fs2.release();  

    //程序结束，输出一些帮助文字
    printf("\n文件读取完毕，请输入任意键结束程序~");
    getchar();

    return 0;  
}  
```



