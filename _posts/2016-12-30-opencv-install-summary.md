---
layout: post
title: Opencv安装总结
date: 2016-12-30
categories: blog
tags: [图像处理]
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

