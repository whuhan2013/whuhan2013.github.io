---
layout: post
title: 手写数字识别实现
date: 2016-12-12
categories: blog
tags: [机器学习]
description: 手写数字识别实现
---

本文主要实现手写数字识别，利用多类逻辑回归与神经网络两种方法实现       

#### Multi-class Classification      

**Dataset**     
There are 5000 training examples in ex3data1.mat, where each training example is a 20 pixel by 20 pixel grayscale image of the digit. Each pixel is represented by a floating point number indicating the grayscale intensity at that location. The 20 by 20 grid of pixels is “unrolled” into a 400-dimensional vector. Each of these training examples becomes a single row in our data matrix X. This gives us a 5000 by 400 matrix X where every row is a training example for a handwritten digit image.     

