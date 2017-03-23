---
layout: post
title: windows下Opencv安装总结    
date: 2017-3-23
categories: blog
tags: [Opencv]
description: Opencv
---

**开发环境**           
windows10+vs2015+Opencv3.1        

**下载vs2015**       
vs2015社区版对个人用户免费，下载之后默认安装即可，需要一定时间才能安装好，选择默认安装后主要是C#的内容，要编写vc++程序，要安装通用Windows平台工具，也需要一定时间，耐心等待即可。      
参见：[Visual studio 2015（VS2015）的下载和安装，以及安装VS2015中的C++](http://blog.csdn.net/quxiaoxia1986/article/details/52352114)    

**下载与安装Opencv3.1**          
在官网下载即可        
点击运行下载好的文件。实际上，opencv的安装程序就是解压缩文件，解压到C盘下Opencv3.1.0文件夹内     

#### 配置        

**OpenCV3.1.0环境变量配置**        
在系统变量的path中加入C:\Opencv3.1.0\opencv\build\x64\vc14\bin         

然后主要步骤如下：         
1、建立一个Win32控制台项目       
2、然后点击视图，在视图下找到其他窗口，在其他窗口下找到属性管理器，点击打开         
3、然后便会有一个属性管理器的窗口了，接下来点开工程文件，下边会有一个Debug|x64的文件夹，点开，下有名为Microsoft.Cpp.x64.user的文件，右键属性           
4、然后选择通用属性下的VC++目录，右边会有包含目录和库目录，点击包含目录，添加以下三条路径，其实这些都是刚才OpenCV相关解压文件所在的目录            
C:\Opencv3.1.0\opencv\build\include            
C:\Opencv3.1.0\opencv\build\include\opencv           
C:\Opencv3.1.0\opencv\build\include\opencv2                
5、.再点击库目录添加下面一条路径             
C:\Opencv3.1.0\opencv\build\x64\vc14\lib           
6、还是刚才的属性页面，点击链接器，选择输入，会在右侧看到附加依赖项，添加下面文件         
opencv_world310d.lib           
这里添加的是Debug模式的，会看到文件的结尾有d，假如要添加Release模式的，将d去掉即可，即opencv_world310.lib           

**测试**        

```
#include <opencv2\opencv.hpp>

using namespace cv;
int main()
{
	Mat picture = imread("p1.jpg");//图片必须添加到工程目录下
										  //也就是和test.cpp文件放在一个文件夹下！！！
	imshow("测试程序", picture);
	waitKey(20150901);
}
```

**注意：**应选择X64模式编译，已不支持X86模式

![](https://raw.githubusercontent.com/whuhan2013/newImage/master/python/chapter1/p5.png)


**参考：**[win10下vs2015配置Opencv3.1.0过程详解(转)](http://www.cnblogs.com/zangdalei/p/5339316.html)



