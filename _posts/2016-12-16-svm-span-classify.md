---
layout: post
title: SVM实现邮件分类
date: 2016-12-16
categories: blog
tags: [机器学习]
description: SVM实现邮件分类
---

首先学习一上svm分类的使用。     

主要有以下步骤：   

- Loading and Visualizing Dataj
- Training Linear SVM    
- Implementing Gaussian Kernel
- Training SVM with RBF Kernel    
- 选择最优的C, sigma参数     
- 画出边界线      

**线性keneral实现**    

```
C = 1;
model = svmTrain(X, y, C, @linearKernel, 1e-3, 20);
visualizeBoundaryLinear(X, y, model);
```

