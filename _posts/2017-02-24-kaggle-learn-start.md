---
layout: post
title: Kaggle学习入门
date: 2017-2-24
categories: blog
tags: [数据分析]
description: 数据分析
---

kaggle是数据挖掘与机器学习领域的常用网站，经常会有各种比赛，适合在数据挖掘与机器学习领域的实战提高。下面以CIFAR-10 - Object Recognition in Images项目为例演示kaggle入门。         

1、下载并解压数据          
直接下载，数据通常比较大，解压需要一定时间,mac下可以使用7za x vps12.7z解压。             

2、读入数据      
通常是csv格式，利用pandas读入数据       

```
df = pd.read_csv("data/train.csv")
```

pandas是python的一个常用库，详情可参见：[Python科学计算(二)](https://www.shiyanlou.com/courses/391)       

3､运行算法        
利用k近邻算法，得到图片分类结果。        

```
import numpy as np
from scipy.misc import imread, imsave, imresize
import pandas as pd

class NearestNeighbor(object):
  def __init__(self):
    pass

  def train(self, X, y):
    """ X is N x D where each row is an example. Y is 1-dimension of size N """
    # the nearest neighbor classifier simply remembers all the training data
    self.Xtr = X
    self.ytr = y

  def predict(self, X):
    """ X is N x D where each row is an example we wish to predict label for """
    num_test = X.shape[0]
    # lets make sure that the output type matches the input type
    Ypred = np.zeros(num_test, dtype = self.ytr.dtype)

    # loop over all test rows
    for i in xrange(num_test):
      # find the nearest training image to the i'th test image
      # using the L1 distance (sum of absolute value differences)
      distances = np.sum(np.abs(self.Xtr - X[i,:]), axis = 1)
      min_index = np.argmin(distances) # get the index with smallest distance
      Ypred[i] = self.ytr[min_index] # predict the label of the nearest example

    return Ypred
```

4、运行算法，得到结果并保存成csv格式        

```
if __name__ == '__main__':
    nearestNeighbor = NearestNeighbor()
    trainSize=500
    testSize=500
    imgMatrix = np.ones((trainSize, 32*32*3))
    testImag = np.ones((testSize,32*32*3))
    for i in xrange(trainSize):
        print i
        img = imread('train/%d.png'%(i+1))
        img_row = img.reshape(1,32*32*3)
        imgMatrix[i]=img_row

    df = pd.read_csv("trainLabels.csv")

    nearestNeighbor.train(imgMatrix,df.label)

    for j in xrange(testSize):
        print j
        img = imread('train/%d.png'%(j+501))
        img_row=img.reshape(1,32*32*3)
        testImag[j]=img_row

    yPred = nearestNeighbor.predict(testImag)
    output = pd.DataFrame(columns=['id','label'])
    output['label']=yPred
    output['id']=range(1,testSize+1)
    output.to_csv('output.csv', index=False)
```

5、将所得到csv文件上传到kaggle，等待分析结果          
使用k近邻算法，所得正确率不高，只有大约30%左右。            
