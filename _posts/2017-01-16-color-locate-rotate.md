---
layout: post
title: 颜色定位与偏斜扭转
date: 2017-1-16
categories: blog
tags: [Opencv]
description: 颜色定位与偏斜扭转
---

在前面的介绍里，我们使用了Sobel查找垂直边缘的方法，成功定位了许多车牌。但是，Sobel法最大的问题就在于面对垂直边缘交错的情况下，无法准确地定位车牌。例如下图。为了解决这个问题，可以考虑使用颜色信息进行定位。              
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p14.jpg)
如果将颜色定位与Sobel定位加以结合的话，可以使车牌的定位准确率从75%上升到94%。     

**方法**     

　　关于颜色定位首先我们想到的解决方案就是：利用RGB值来判断。

　　这个想法听起来很自然：如果我们想找出一幅图像中的蓝色部分，那么我们只需要检查RGB分量（RGB分量由Red分量--红色，Green分量--绿色，Blue分量--蓝色共同组成）中的Blue分量就可以了。一般来说，Blue分量是个0到255的值。如果我们设定一个阈值，并且检查每个像素的Blue分量是否大于它，那我们不就可以得知这些像素是不是蓝色的了么？这个想法虽然很好，不过存在一个问题，我们该怎么来选择这个阈值？这是第一个问题。

　　即便我们用一些方法决定了阈值以后，那么下面的一个问题就会让人抓狂，颜色是组合的，即便蓝色属性在255（这样已经很‘蓝’了吧），只要另外两个分量配合（例如都为255），你最后得到的不是蓝色，而是黑色。

　　这还只是区分蓝色的问题，黄色更麻烦，它是由红色和绿色组合而成的，这意味着你需要考虑两个变量的配比问题。这些问题让选择RGB颜色作为判断的难度大到难以接受的地步。因此必须另想办法。

　　为了解决各种颜色相关的问题，人们发明了各种颜色模型。其中有一个模型，非常适合解决颜色判断的问题。这个模型就是HSV模型。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p15.jpg)
HSV模型是根据颜色的直观特性创建的一种圆锥模型。与RGB颜色模型中的每个分量都代表一种颜色不同的是，HSV模型中每个分量并不代表一种颜色，而分别是：色调（H），饱和度（S），亮度（V）。

　　H分量是代表颜色特性的分量，用角度度量，取值范围为0～360，从红色开始按逆时针方向计算，红色为0，绿色为120，蓝色为240。S分量代表颜色的饱和信息，取值范围为0.0～1.0，值越大，颜色越饱和。V分量代表明暗信息，取值范围为0.0～1.0，值越大，色彩越明亮。

　　H分量是HSV模型中唯一跟颜色本质相关的分量。只要固定了H的值，并且保持S和V分量不太小，那么表现的颜色就会基本固定。为了判断蓝色车牌颜色的范围，可以固定了S和V两个值为1以后，调整H的值，然后看颜色的变化范围。通过一段摸索，可以发现当H的取值范围在200到280时，这些颜色都可以被认为是蓝色车牌的颜色范畴。于是我们可以用H分量是否在200与280之间来决定某个像素是否属于蓝色车牌。黄色车牌也是一样的道理，通过观察，可以发现当H值在30到80时，颜色的值可以作为黄色车牌的颜色。

光判断H分量的值是否就足够了？

　　事实上是不足的。固定了H的值以后，如果移动V和S会带来颜色的饱和度和亮度的变化。当V和S都达到最高值，也就是1时，颜色是最纯正的。降低S，颜色越发趋向于变白。降低V，颜色趋向于变黑，当V为0时，颜色变为黑色。因此，S和V的值也会影响最终颜色的效果。      
我们知道H分量基本能表示一个物体的颜色，但是S和V的取值也要在一定范围内，因为S代表的是H所表示的那个颜色和白色的混合程度，也就说S越小，颜色越发白，也就是越浅；V代表的是H所表示的那个颜色和黑色的混合程度，也就说V越小，颜色越发黑。

　　我们可以设置一个阈值，假设S和V都大于阈值时，颜色才属于H所表达的颜色。

　　在EasyPR里，这个值是0.35，也就是V属于0.35到1且S属于0.35到1的一个范围，类似于一个矩形。对V和S的阈值判断是有必要的，因为很多车牌周身的车身，都是H分量属于200-280，而V分量或者S分量小于0.35的。通过S和V的判断可以排除车牌周围车身的干扰。

明确了使用HSV模型以及用阈值进行判断以后，下面就是一个颜色定位的完整过程。

- 第一步，将图像的颜色空间从RGB转为HSV，在这里由于光照的影响，对于图像使用直方图均衡进行预处理；
- 第二步，依次遍历图像的所有像素，当H值落在200-280之间并且S值与V值也落在0.35-1.0之间，标记为白色像素，否则为黑色像素；
- 第三步，对仅有白黑两个颜色的二值图参照原先车牌定位中的方法，使用闭操作，取轮廓等方法将车牌的外接矩形截取出来做进一步的处理。

以上就完成了一个蓝色车牌的定位过程。我们把对图像中蓝色车牌的寻找过程称为一次与蓝色模板的匹配过程。代码中的函数称之为colorMatch。一般说来，一幅图像需要进行一次蓝色模板的匹配，还要进行一次黄色模板的匹配，以此确保蓝色和黄色的车牌都被定位出来。

　　黄色车牌的定位方法与其类似，仅仅只是H阈值范围的不同。事实上，黄色定位的效果一般好的出奇，可以在非常复杂的环境下将车牌极为准确的定位出来，这可能源于现实世界中黄色非常醒目的原因。

从实际效果来看，颜色定位的效果是很好的。在通用数据测试集里，大约70%的车牌都可以被定位出来（一些颜色定位不了的，我们可以用Sobel定位处理）。

　　在代码中有些细节需要注意：

　　一. opencv为了保证HSV三个分量都落在0-255之间（确保一个char能装的下），对H分量除以了2，也就是0-180的范围，S和V分量乘以了255，将0-1的范围扩展到0-255。我们在设置阈值的时候需要参照opencv的标准，因此对参数要进行一个转换。

　　二. 是v和s取值的问题。对于暗的图来说，取值过大容易漏，而对于亮的图，取值过小则容易跟车身混淆。因此可以考虑最适应的改变阈值。

　　三. 是模板问题。目前的做法是针对蓝色和黄色的匹配使用了两个模板，而不是统一的模板。统一模板的问题在于担心蓝色和黄色的干扰问题，例如黄色的车与蓝色的牌的干扰，或者蓝色的车和黄色牌的干扰，这里面最典型的例子就是一个带有蓝色车牌的黄色出租车，在很多城市里这已经是“标准配置”。因此需要将蓝色和黄色的匹配分别用不同的模板处理。

　　了解完这三个细节以后，下面就是代码部分

```
//! 根据一幅图像与颜色模板获取对应的二值图
    //! 输入RGB图像, 颜色模板（蓝色、黄色）
    //! 输出灰度图（只有0和255两个值，255代表匹配，0代表不匹配）
    Mat colorMatch(const Mat& src, Mat& match, const Color r, const bool adaptive_minsv)
    {
        // S和V的最小值由adaptive_minsv这个bool值判断
        // 如果为true，则最小值取决于H值，按比例衰减
        // 如果为false，则不再自适应，使用固定的最小值minabs_sv
        // 默认为false
        const float max_sv = 255;
        const float minref_sv = 64;

        const float minabs_sv = 95;

        //blue的H范围
        const int min_blue = 100;  //100
        const int max_blue = 140;  //140

        //yellow的H范围
        const int min_yellow = 15; //15
        const int max_yellow = 40; //40

        Mat src_hsv;
        // 转到HSV空间进行处理，颜色搜索主要使用的是H分量进行蓝色与黄色的匹配工作
        cvtColor(src, src_hsv, CV_BGR2HSV);

        vector<Mat> hsvSplit;
        split(src_hsv, hsvSplit);
        equalizeHist(hsvSplit[2], hsvSplit[2]);
        merge(hsvSplit, src_hsv);

        //匹配模板基色,切换以查找想要的基色
        int min_h = 0;
        int max_h = 0;
        switch (r) {
        case BLUE:
            min_h = min_blue;
            max_h = max_blue;
            break;
        case YELLOW:
            min_h = min_yellow;
            max_h = max_yellow;
            break;
        }

        float diff_h = float((max_h - min_h) / 2);
        int avg_h = min_h + diff_h;

        int channels = src_hsv.channels();
        int nRows = src_hsv.rows;
        //图像数据列需要考虑通道数的影响；
        int nCols = src_hsv.cols * channels;

        if (src_hsv.isContinuous())//连续存储的数据，按一行处理
        {
            nCols *= nRows;
            nRows = 1;
        }

        int i, j;
        uchar* p;
        float s_all = 0;
        float v_all = 0;
        float count = 0;
        for (i = 0; i < nRows; ++i)
        {
            p = src_hsv.ptr<uchar>(i);
            for (j = 0; j < nCols; j += 3)
            {
                int H = int(p[j]); //0-180
                int S = int(p[j + 1]);  //0-255
                int V = int(p[j + 2]);  //0-255

                s_all += S;
                v_all += V;
                count++;

                bool colorMatched = false;

                if (H > min_h && H < max_h)
                {
                    int Hdiff = 0;
                    if (H > avg_h)
                        Hdiff = H - avg_h;
                    else
                        Hdiff = avg_h - H;

                    float Hdiff_p = float(Hdiff) / diff_h;

                    // S和V的最小值由adaptive_minsv这个bool值判断
                    // 如果为true，则最小值取决于H值，按比例衰减
                    // 如果为false，则不再自适应，使用固定的最小值minabs_sv
                    float min_sv = 0;
                    if (true == adaptive_minsv)
                        min_sv = minref_sv - minref_sv / 2 * (1 - Hdiff_p); // inref_sv - minref_sv / 2 * (1 - Hdiff_p)
                    else
                        min_sv = minabs_sv; // add

                    if ((S > min_sv && S < max_sv) && (V > min_sv && V < max_sv))
                        colorMatched = true;
                }

                if (colorMatched == true) {
                    p[j] = 0; p[j + 1] = 0; p[j + 2] = 255;
                }
                else {
                    p[j] = 0; p[j + 1] = 0; p[j + 2] = 0;
                }
            }
        }

        //cout << "avg_s:" << s_all / count << endl;
        //cout << "avg_v:" << v_all / count << endl;

        // 获取颜色匹配后的二值灰度图
        Mat src_grey;
        vector<Mat> hsvSplit_done;
        split(src_hsv, hsvSplit_done);
        src_grey = hsvSplit_done[2];

        match = src_grey;

        return src_grey;
    }
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p16.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p17.png)

**不足**             
　　以上说明了颜色定位的设计思想与细节。那么颜色定位是不是就是万能的？答案是否定的。在色彩充足，光照足够的情况下，颜色定位的效果很好，但是在面对光线不足的情况，或者蓝色车身的情况时，颜色定位的效果很糟糕。下图是一辆蓝色车辆，可以看出，车牌与车身内容完全重叠，无法分割.        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p18.jpg)
碰到失效的颜色定位情况时需要使用原先的Sobel定位法。

　　目前的新版本使用了颜色定位与Sobel定位结合的方式。首先进行颜色定位，然后根据条件使用Sobel进行再次定位，增加整个系统的适应能力。

　　为了加强鲁棒性，Sobel定位法可以用两阶段的查找。也就是在已经被Sobel定位的图块中，再进行一次Sobel定位。这样可以增加准确率，但会降低了速度。一个折衷的方案是让用户决定一个参数m_maxPlates的值，这个值决定了你在一幅图里最多定位多少车牌。系统首先用颜色定位出候选车牌，然后通过SVM模型来判断是否是车牌，最后统计数量。如果这个数量大于你设定的参数，则认为车牌已经定位足够了，不需要后一步处理，也就不会进行两阶段的Sobel查找。相反，如果这个数量不足，则继续进行Sobel定位。

　　综合定位的代码位于CPlateDectec中的的成员函数plateDetectDeep中，以下是plateDetectDeep的整体流程
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p19.jpg)  
有没有颜色定位与Sobel定位都失效的情况？有的。这种情况下可能需要使用第三类定位技术--字符定位技术。这是EasyPR发展的一个方向，这里不展开讨论。

### 偏斜扭转       
　　解决了颜色的定位问题以后，下面的问题是：在定位以后，我们如何把偏斜过来的车牌扭正呢？    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p20.jpg)    
　这个过程叫做偏斜扭转过程。其中一个关键函数就是opencv的仿射变换函数。但在具体实施时，有很多需要解决的问题。

我们要解决的问题包括三类，第一类是正的车牌，第二类是倾斜的车牌，第三类是偏斜的车牌。前两类是前面说过的，第三类是本次新增的功能需求。第二类倾斜车牌与第三类车牌的区别见下图。    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p21.jpg)  
通过上图可以看出，正视角的旋转图片的观察角度仍然是正方向的，只是由于路的不平或者摄像机的倾斜等原因，导致矩形有一定倾斜。这类图块的特点就是在RotataedRect内部，车牌部分仍然是个矩形。偏斜视角的图片的观察角度是非正方向的，是从侧面去看车牌。这类图块的特点是在RotataedRect内部，车牌部分不再是个矩形，而是一个平行四边形。这个特性决定了我们需要区别的对待这两类图片。

　　一个初步的处理思路就是下图。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/carplate/p22.jpg)  
简单来说，整个处理流程包括下面四步：

1.感兴趣区域的截取     
2.角度判断       
3.偏斜判断       
4.仿射变换         

　　接下来按照这四个步骤依次介绍       

**ROI截取**

　　如果要使用区域旋转，首先我们必须从原图中截取出一个包含定位区域的图块。

　　opencv提供了一个从图像中截取感兴趣区域ROI的方法，也就是Mat(Rect ...)。这个方法会在Rect所在的位置，截取原图中一个图块，然后将其赋值到一个新的Mat图像里。遗憾的是这个方法不支持RotataedRect，同时Rect与RotataedRect也没有继承关系。因此也不能直接调用这个方法。

　　我们可以使用RotataedRect的boudingRect()方法。这个方法会返回一个RotataedRect的最小外接矩形，而且这个矩形是一个Rect。因此将这个Rect传递给Mat(Rect...)方法就可以截取出原图的ROI图块，并获得对应的ROI图像。

　　需要注意的是，ROI图块和ROI图像的区别，当我们给定原图以及一个Rect时，原图中被Rect包围的区域称为ROI图块，此时图块里的坐标仍然是原图的坐标。当这个图块里的内容被拷贝到一个新的Mat里时，我们称这个新Mat为ROI图像。ROI图像里仅仅只包含原来图块里的内容，跟原图没有任何关系。所以图块和图像虽然显示的内容一样，但坐标系已经发生了改变。在从ROI图块到ROI图像以后，点的坐标要计算一个偏移量。

　　下一步的工作中可以仅对这个ROI图像进行处理，包括对其旋转或者变换等操作。

　　示例图片中的截取出来的ROI图像如下图：






