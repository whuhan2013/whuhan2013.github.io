---
layout: post
title: K近邻分类器python实现
date: 2017-2-24
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

本作业的目标如下：

- 理解基本的图像分类流程和数据驱动方法（训练与预测阶段）。
- 理解训练、验证、测试分块，学会使用验证数据来进行超参数调优。
- 熟悉使用numpy来编写向量化代码。
- 实现并应用k-最近邻（k-NN）分类器。
- 实现并应用支持向量机（SVM）分类器。
- 实现并应用Softmax分类器。
- 实现并应用一个两层神经网络分类器。
- 理解以上分类器的差异和权衡之处。
- 基本理解使用更高层次表达相较于使用原始图像像素对算法性能的提升（例如：色彩直方图和梯度直方图HOG）。

推荐安装[Anaconda](http://link.zhihu.com/?target=https%3A//www.continuum.io/downloads),下载sh版本后，bash安装，source .bash_profile即可，会将系统默认的python修改成我们的，Anaconda是Python的一个发布版，包含了最流行的科研、数学、工程和数据分析Python包。

IPython notebook的使用，参见教程：[http://cs231n.github.io/ipython-tutorial/](http://cs231n.github.io/ipython-tutorial/)

#### KNN分类器实现         

首先读取数据      

```
import cPickle as pickle
import numpy as np
import os
from scipy.misc import imread

def load_CIFAR_batch(filename):
  """ load single batch of cifar """
  with open(filename, 'rb') as f:
    datadict = pickle.load(f)
    X = datadict['data']
    Y = datadict['labels']
    X = X.reshape(10000, 3, 32, 32).transpose(0,2,3,1).astype("float")
    Y = np.array(Y)
    return X, Y

def load_CIFAR10(ROOT):
  """ load all of cifar """
  xs = []
  ys = []
  for b in range(1,6):
    f = os.path.join(ROOT, 'data_batch_%d' % (b, ))
    X, Y = load_CIFAR_batch(f)
    xs.append(X)
    ys.append(Y)    
  Xtr = np.concatenate(xs)
  Ytr = np.concatenate(ys)
  del X, Y
  Xte, Yte = load_CIFAR_batch(os.path.join(ROOT, 'test_batch'))
  return Xtr, Ytr, Xte, Yte
```

transpose函数解释：arr1.shape 应该是(2, 2, 4) 意为 2维，2*4矩阵

arr1.transpose(*args) 里面的参数，可以这么理解，他是调换arr1.shape的顺序，咱来给arr1.shape标一下角标哈，（2[0], 2[1], 4[2]）  [ ] 里是shape的索引，对吧， 
transpose((1, 0, 2)) 的意思是 按照这个顺序 重新设置shape 也就是 （2[1], 2[0], 4[2]）

虽然看起来 变换前后的shape都是 2,2,4  ， 但是问题来了，transpose是转置
shape按照(1,0,2)的顺序重新设置了， array里的所有元素 也要按照这个规则重新组成新矩阵

比如 8 在arr1中的索引是 (1, 0, 0)  那么按照刚才的变换规则，就是 (0, 1, 0) 看看跟你结果arr2的位置一样了吧，依此类推...

concatenate函数解释：用于合并数组，在这里用于将一个list里的array合并成一个array。        

**画图**       

```
classes = ['plane', 'car', 'bird', 'cat', 'deer', 'dog', 'frog', 'horse', 'ship', 'truck']
num_classes = len(classes)
samples_per_class = 7
for y, cls in enumerate(classes):
    idxs = np.flatnonzero(y_train == y)
    idxs = np.random.choice(idxs, samples_per_class, replace=False)
    for i, idx in enumerate(idxs):
        plt_idx = i * num_classes + y + 1
        plt.subplot(samples_per_class, num_classes, plt_idx)
        plt.imshow(X_train[idx].astype('uint8'))
        plt.axis('off')
        if i == 0:
            plt.title(cls)
plt.show();
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/p6.jpg)

**计算图像间距离的三种方法**        

```
for i in xrange(num_test):
      for j in xrange(num_train):
        #print self.X_train[i].shape,X[j].shape
        distances = np.sqrt(np.sum(np.square(self.X_train[j] - X[i])))
        #print distances
        dists[i,j]=distances

for i in xrange(num_test):
      dists[i]=np.sqrt(np.sum(np.square(self.X_train - X[i,:]), axis = 1))

dists = np.sqrt(((np.sum(self.X_train*self.X_train,axis=1)-np.dot(X,self.X_train.T)*2).T+np.sum(X*X,axis=1)).T)
```

分别为用两个循环，一个循环，不用循环实现。        
从效率上看           
Two loop version took 65.320027 seconds         
One loop version took 65.133510 seconds          
No loop version took 0.526674 seconds       


**预测**        

```
closest_x=[]
      closest_y = []
      min_index = dists[i].argsort()[0]
      closest_x.append(dists[i].argsort()[0:k])
    
      closest_y=self.y_train[closest_x[:]]
      #print type(closest_y)
      closest_y=list(closest_y)
      #print closest_y
      myset = set(closest_y)  
      max=0
      for item in myset:
            if closest_y.count(item)>max:
                max=closest_y.count(item)
                y_pred[i]=item

    return y_pred
```


**交叉验证**       

```
num_folds = 5
k_choices = [1, 3, 5, 8, 10, 12, 15, 20, 50, 100]

X_train_folds = []
y_train_folds = []

X_train_folds = np.array_split(X_train, num_folds)
y_train_folds = np.array_split(y_train, num_folds)


# A dictionary holding the accuracies for different values of k that we find
# when running cross-validation. After running cross-validation,
# k_to_accuracies[k] should be a list of length num_folds giving the different
# accuracy values that we found when using that value of k.
k_to_accuracies = {}
def getTrainFold(train_folds,k):
    result = []
    for i in range(num_folds):
        if i != k:
            result.append(train_folds[i])
    return np.concatenate(result)

for k in k_choices:
    temp = range(num_folds)
    accuracies_length = []
    for i in range(num_folds):
        print i
        mX_train = getTrainFold(X_train_folds,i)
        
        mX_train = np.reshape(mX_train, (mX_train.shape[0], -1))
        mX_test = np.reshape(X_train_folds[i], (X_train_folds[i].shape[0], -1))
        classifier.train(mX_train, getTrainFold(y_train_folds,i))
        
        #print X_train_folds[i].shape             
        m=classifier.compute_distances_no_loops(mX_test)
        my_test_pred = classifier.predict_labels(m, k)
        mnum_correct = np.sum(my_test_pred == y_train_folds[i])
        maccuracy = float(mnum_correct) / mX_test[i].shape[0]
        accuracies_length.append(maccuracy)    
    k_to_accuracies[k]=accuracies_length 
    
    


# Print out the computed accuracies
for k in sorted(k_to_accuracies):
    for accuracy in k_to_accuracies[k]:
        print 'k = %d, accuracy = %f' % (k, accuracy)
```


**最后选择最佳的k，求得最高的准确率**       

```
best_k = 1

classifier = KNearestNeighbor()
classifier.train(X_train, y_train)
y_test_pred = classifier.predict(X_test, k=best_k)

# Compute and display the accuracy
num_correct = np.sum(y_test_pred == y_test)
accuracy = float(num_correct) / num_test
print 'Got %d / %d correct => accuracy: %f' % (num_correct, num_test, accuracy)
```


