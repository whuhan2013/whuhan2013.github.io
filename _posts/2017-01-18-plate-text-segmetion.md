---
layout: post
title: 车牌识别之字符分割
date: 2017-1-18
categories: blog
tags: [Opencv]
description: 字符分割
---

由于在车牌定位中，我们使用了归一化过程。因此所需要处理的车牌的大小是统一的，在目前的版本中这个值是136*36。

　　那么字符分割的结果就是将车牌中的所有文字一一分割开来，形成单一的字符块。生成的字符块就可以输入下一步的字符识别部分进行识别。我们可以使用人工神经网络，也就是ANN来识别字符。

　　具体而言，字符分割过程是如何做的呢？简单说，就是：灰度化->颜色判断->二值化->取轮廓->找外接矩形->截取图块。
![](http://images0.cnblogs.com/blog2015/673793/201507/271448349061225.jpg)

**1.灰度化**       
　　首先，我们把彩色的图片转化为灰度化图片。注意：为了以后可以利用彩色信息，在前面的车牌检测过程中，我们的输出结果不是灰度化图片，而是彩色图片。这样以后当我们改正算法，想利用彩色信息时就可以使用了。

　　但是在这里，我们的算法还是针对的是灰度化图片，因此首先进行灰度化处理。

　　灰度化后的图片见下图：
![](http://images0.cnblogs.com/blog2015/673793/201507/251509096563831.jpg)     

**2.颜色判断**                   
　　灰度化之后，为了分割字符。我们需要获取字符的轮廓。注意：分割字符有很多种方法。例如投影法，滑动窗口判断法，在这里，使用的是取字符轮廓法。

　　因为需要取轮廓，就需要把图片转化成一个二值化图片。不过，由于蓝色和黄色车牌图片的区别，两者需要用的二值化参数不一样，因此这里需要对车牌图片的颜色进行一个判断。车牌颜色对二值化的影响的分析见后面“其他细节”章节。

　　这里颜色判断的使用的是前面颜色定位详解里的模板匹配法。
![](http://images0.cnblogs.com/blog2015/673793/201507/271501430162052.jpg)   

**3.二值化**     
　　获取颜色后，就可以选择不同的参数进行大津阈值法来进行二值化。对于本示例图片中的蓝色车牌而言，使用的参数为CV_THRESH_BINARY。

　　二值化后的效果见下图：      
![](http://images0.cnblogs.com/blog2015/673793/201507/251532116561889.jpg)        

**4.取轮廓**        
　　接下来，使用被多次用到的取轮廓方法findContours。关于这个方法的具体内容，在前面的开发详解中已做过介绍，这里不再赘述。

　　取轮廓后的结果如下图：
![](http://images0.cnblogs.com/blog2015/673793/201507/251604550467627.jpg)  
注意：直接使用findContours方法取轮廓时，在处理中文字符，也就是“苏”时，会发生断裂现象。因此为了处理中文字符，EasyPR换了一种思路，使用了额外的步骤来解决这个问题。具体可以见后面的“中文字符处理”章节。         

**5.找外接矩形**       
　　使用了中文字符处理方法以后，成功获取了所有的字符的外接矩形。

　　具体见下图：
![](http://images0.cnblogs.com/blog2015/673793/201507/251606213598493.jpg)        

**6.截取图块**      
　　最后，把图中的外接矩形一一截取出来，归一化到统一格式。留待输入下个步骤--字符识别模块处理。

　　归一化后字符图块见下图：
![](http://images0.cnblogs.com/blog2015/673793/201507/251612306257647.jpg)     

#### 中文字符处理
　　上面的流程在处理英文车牌时，效果是很好的。但是在处理中文车牌时，存在一个很大的问题。

　　在取轮廓时，中文由于自身的特性，例如有笔画区间，取轮廓会造成断裂现象。例如下图中的“苏”。英文字符通过取轮廓都被完整的包括了，而“苏”字则分成了两个连通区域。
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251044155157380.jpg"></center>
虽然并不是所有的中文都会存在这个问题（例如下图的“津”字），但直接用取轮廓操作已经不合适了。

　　那么如何解决这个问题的呢？其实想法很简单。那就是既然有些中文字符没办法用取轮廓处理，那么就干脆先不处理中文字符，而是用取轮廓操作处理中文字符后面的字符。例如“苏A88M88”，其中“A88M88”这六个字符我都能用取轮廓操作获得。我先获取这六个字符，再想办法获取中文字符。            
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/272107332817556.jpg"></center>
获取这六个字符后，接下来该如何获取“苏”这个中文字符的轮廓呢？

　　这里的关键就是“苏”字符后面的“A”字符，这个字符在中文车牌里代表城市的代码，我们在这里简称它为“城市字符”或者“特殊字符”。

　　这个字符有一个特征，就是与后面的字符存在一定的间隔。但是与前面的中文字符靠的较紧。倘若我获取了这个特殊字符的外接矩形，只要把这个外接矩形向左做一些的偏移（偏移的大小可以通过经验指定，例如设置为字符宽度的1.15倍），这样这个外接矩形就成了包含中文字符的一个矩形了。下面就可以截取中文字符的图块。

　　下图就是“特殊字符”与被反推得到的“中文字符”的矩形，在图中用红色矩形表示。     
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251212013124250.jpg"></center>
下面的问题就是如何获取“特殊字符”的位置？

　　一种方法是把所有取轮廓操作获取到的矩形进行排序，最左边的就是特殊字符的图块。但是有些中文字符会被取轮廓操作截取为一个连通区域。在这种情况下，最左边的图块矩形是中文字符的矩形，而不是特殊字符的矩形了。所以这个方法不能用。

　　另一种方法就是依次判断所有取轮廓操作得到的矩形的位置，设矩形的中点恰好在整个车牌的1/7到2/7之间时的矩形为特殊矩形。这样操作的前提是我们的车牌定位的非常准确，恰到把整个车牌截取的正正好。在这种情况下，只要外接矩形满足这些条件，就可以判断为特殊字符的矩形。

　　这个方法思路很简单，实际中应用效果也不错，因此也是我们目前采用的方法。      
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251652390934512.jpg"></center>       

```
//! 找出指示城市的字符的Rect，例如苏A7003X，就是"A"的位置
int CCharsSegment::GetSpecificRect(const vector<Rect>& vecRect) {
  vector<int> xpositions;
  int maxHeight = 0;
  int maxWidth = 0;

  for (size_t i = 0; i < vecRect.size(); i++) {
    xpositions.push_back(vecRect[i].x);

    if (vecRect[i].height > maxHeight) {
      maxHeight = vecRect[i].height;
    }
    if (vecRect[i].width > maxWidth) {
      maxWidth = vecRect[i].width;
    }
  }

  int specIndex = 0;
  for (size_t i = 0; i < vecRect.size(); i++) {
    Rect mr = vecRect[i];
    int midx = mr.x + mr.width / 2;

    //如果一个字符有一定的大小，并且在整个车牌的1/7到2/7之间，则是我们要找的特殊字符
    //当前字符和下个字符的距离在一定的范围内
    if ((mr.width > maxWidth * 0.8 || mr.height > maxHeight * 0.8) &&
        (midx < int(m_theMatWidth / 7) * 2 &&
         midx > int(m_theMatWidth / 7) * 1)) {
      specIndex = i;
    }
  }

  return specIndex;
}
```

以上就是EasyPR能处理中文车牌的主要原因。原先的taotao1233的代码中无法处理中文的原因就是没有这样一步预处理。其实这是一个很简单的思想，但在之前并没有被实现。EasyPR里实现了这个思路，同时发现，这个方法效果出奇的好。基本可以应对所有的情况。所以说，这个方法可以说是一个简单，有效的处理中文车牌的方法.            

#### 其他一些细节

**1.颜色判断**     
　　在进行二值化前，需要进行一次颜色判断，这是因为对于蓝色和黄色车牌而言，使用的二值化策略必须不同。
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251706134848778.jpg"></center>
对于蓝色车牌而言，使用的参数为CV_THRESH_BINARY。

而对于黄色车牌而言，使用的参数为CV_THRESH_BINARY_INV。

　　假设黄色车牌使用了CV_THRESH_BINARY作为参数，则会发生如下图一样的二值化结果，其中字符部分变成了黑色，而背景则是白色（同理，蓝色车牌使用CV_THRESH_BINARY_INV也是一样的效果）。

　　在这种不正确的参数带来的二值化情况下，取轮廓操作将无法按照预期的行为进行处理。因此，必须使用正确的二值化参数。
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251714140629792.jpg"></center>
在颜色判断时，有一个小技巧，就是先把四周的“边”截取后再进行颜色的判断，这样可以消除车牌定位时一些多余的四周的干扰

```
1   Mat tmpMat = input(Rect_<double>(w * 0.1, h * 0.1, w * 0.8, h * 0.8));
2 
3   // 判断车牌颜色以此确认threshold方法
4   Color plateType = getPlateType(tmpMat, true);
```

颜色判断代码如下

```
// getPlateType
//判断车牌的类型
Color getPlateType(const Mat& src, const bool adaptive_minsv) {
  float max_percent = 0;
  Color max_color = UNKNOWN;

  float blue_percent = 0;
  float yellow_percent = 0;
  float white_percent = 0;

  if (plateColorJudge(src, BLUE, adaptive_minsv, blue_percent) == true) {
    // cout << "BLUE" << endl;
    return BLUE;
  } else if (plateColorJudge(src, YELLOW, adaptive_minsv, yellow_percent) ==
             true) {
    // cout << "YELLOW" << endl;
    return YELLOW;
  } else if (plateColorJudge(src, WHITE, adaptive_minsv, white_percent) ==
             true) {
    // cout << "WHITE" << endl;
    return WHITE;
  } else {
    // cout << "OTHER" << endl;

    // 如果任意一者都不大于阈值，则取值最大者
    max_percent = blue_percent > yellow_percent ? blue_percent : yellow_percent;
    max_color = blue_percent > yellow_percent ? BLUE : YELLOW;

    max_color = max_percent > white_percent ? max_color : WHITE;
    return max_color;
  }
}
```

**2.排除缝隙**      
　　在获得中文字符图块以后，下面一步就是把剩下的图块获取了。不过由于中文车牌一般只有7个字符，所以可以把后面的图块从左到右排序，依次选择6个即可。一些会被误判为“I”的缝隙可以通过这种方法排除出去。

　　例如下图中，最右边的一个缝隙会被误识别为"1"。但是倘若从左到右依次选择的话，这个缝隙并不会被选入候选集合中，因为它已经是“第八个”字符了。
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251117077036800.jpg"></center>

```
//! 这个函数做两个事情
//  1.把特殊字符Rect左边的全部Rect去掉，后面再重建中文字符的位置。
//  2.从特殊字符Rect开始，依次选择6个Rect，多余的舍去。
int CCharsSegment::RebuildRect(const vector<Rect>& vecRect,
                               vector<Rect>& outRect, int specIndex) {
  int count = 6;
  for (size_t i = specIndex; i < vecRect.size() && count; ++i, --count) {
    outRect.push_back(vecRect[i]);
  }

  return 0;
}
```

**3.去除柳钉**    
　　有些中国的车牌中有一个非常妨碍识别的东西，那就是柳钉。倘若对一副含有柳钉的图进行二值化，极有可能会出现下图的结果。一些字符图块（下图的"9"和"1"）通过柳钉的原因联系到了一体，那样的话就无法通过取轮廓操作来分割了。
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251117489685725.jpg"></center>
因此在二值化之后，还需要一个去除柳钉的操作。

　　去除柳钉的思想也并不复杂，就是依次扫描每行，判断跳变次数。车牌字符所在的行的跳变次数是很多的，而柳钉所在的行就会偏少。因此当发现某行跳变次数较少，则可以把该行的所有像素值赋值为0，这样就会大幅度消除柳钉的影响了。

　　下图就是去除柳钉后的效果。
<center><img src="http://images0.cnblogs.com/blog2015/673793/201507/251117586404804.jpg"></center>

```
//去除车牌上方的钮钉
//计算每行元素的阶跃数，如果小于X认为是柳丁，将此行全部填0（涂黑）
// X的推荐值为，可根据实际调整
bool clearLiuDing(Mat& img) {
  vector<float> fJump;
  int whiteCount = 0;
  const int x = 7;
  Mat jump = Mat::zeros(1, img.rows, CV_32F);
  for (int i = 0; i < img.rows; i++) {
    int jumpCount = 0;

    for (int j = 0; j < img.cols - 1; j++) {
      if (img.at<char>(i, j) != img.at<char>(i, j + 1)) jumpCount++;

      if (img.at<uchar>(i, j) == 255) {
        whiteCount++;
      }
    }

    jump.at<float>(i) = (float)jumpCount;
  }

  int iCount = 0;
  for (int i = 0; i < img.rows; i++) {
    fJump.push_back(jump.at<float>(i));
    if (jump.at<float>(i) >= 16 && jump.at<float>(i) <= 45) {
      //车牌字符满足一定跳变条件
      iCount++;
    }
  }

  ////这样的不是车牌
  if (iCount * 1.0 / img.rows <= 0.40) {
    //满足条件的跳变的行数也要在一定的阈值内
    return false;
  }
  //不满足车牌的条件
  if (whiteCount * 1.0 / (img.rows * img.cols) < 0.15 ||
      whiteCount * 1.0 / (img.rows * img.cols) > 0.50) {
    return false;
  }

  for (int i = 0; i < img.rows; i++) {
    if (jump.at<float>(i) <= x) {
      for (int j = 0; j < img.cols; j++) {
        img.at<char>(i, j) = 0;
      }
    }
  }
  return true;
}
```

**总结**  

　　最后回顾一下整体的处理流程，首先是对车牌图像进行灰度化，然后根据车牌的不同颜色来进行不同的二值化处理。二值化完后首先去除柳钉，然后进行取轮廓操作。

　　取轮廓操作以后，在所有的轮廓中根据先验知识，找到代表城市的字符，也就是“苏A”中“A”的位置，根据“A”的位置来反推“苏”的位置。

　　最后将找到的这些轮廓依次排序，从左到右依次选择6个，和第一个的中文字符组成7个字符的图块数组，输入到下一步字符识别模块中进行处理。

　　整个字符分割流程就到此结束了，还是比较简单的。其中的中文字符位置的确定使用了“先验知识”这种方法。这种方法在面对固定已知场景中是较好的方法，但是面对特殊情况时就可能会有不太好的效果，因此要根据具体情况来权衡。    


