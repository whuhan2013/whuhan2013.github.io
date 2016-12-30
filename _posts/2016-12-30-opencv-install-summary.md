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
include "opencv2/opencv.hpp"

int main()
{
    cv::Mat img = cv::imread("/Users/schuser/beauty.jpg");
    cv::imshow("Image", img);
    cv::waitKey(0);

    return 0;
}
```



