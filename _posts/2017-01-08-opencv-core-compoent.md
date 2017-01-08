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


