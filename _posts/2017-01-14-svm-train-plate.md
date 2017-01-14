---
layout: post
title: SVM训练车牌
date: 2017-1-14
categories: blog
tags: [Opencv]
description: 车牌判断
---


我们已经知道，车牌定位模块的输出是一些候选车牌的图片。但如何从这些候选车牌图片中甄选出真正的车牌，就是通过SVM模型判断/预测得到的。   

　简单来说，EasyPR的车牌判断模块就是将候选车牌的图片一张张地输入到SVM模型中，然后问它，这是车牌么？如果SVM模型回答不是，那么就继续下一张，如果是，则把图片放到一个输出列表里。最后把列表输入到下一步处理。由于EasyPR使用的是列表作为输出，因此它可以输出一副图片中所有的车牌，不像一些车牌识别程序，只能输出一个车牌结果。

现在，让我们一步步地，进入这个SVM模型的核心看看，它是如何做到判断一副图片是车牌还是不是车牌的？本文主要分为三个大的部分：

- SVM应用：描述如何利用SVM模型进行车牌图片的判断。
- SVM训练：说明如何通过一系列步骤得到SVM模型。
- SVM调优：讨论如何对SVM模型进行优化，使其效果更加好。  

#### SVM应用      
我们在SVM模型中一开始输入的原始信息也是图像的所有像素，然后SVM模型通过对这些像素进行分析，输出这个图片是否是车牌的结论。     

```
 int PlateJudge::plateJudge(const Mat &inMat, int &result) {
    Mat features;
    extractFeature(inMat, features);

    float response = svm_->predict(features);
    /*std::cout << "response:" << response << std::endl;

    float score = svm_->predict(features, noArray(), cv::ml::StatModel::Flags::RAW_OUTPUT);
    std::cout << "score:" << score << std::endl;*/

    result = (int)response;

    return 0;
  }
```








 