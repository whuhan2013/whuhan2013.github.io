---
layout: post
title: Softmax分类器python实现
date: 2017-3-11
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

本文主要包括以下内容：        

- implement a fully-vectorized loss function for the Softmax classifier
- implement the fully-vectorized expression for its analytic gradient
- check your implementation with numerical gradient
- use a validation set to tune the learning rate and regularization strength
- optimize the loss function with SGD
- visualize the final learned weights

**计算损失函数**           
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter9/p1.png)
详细可参见：[Softmax损失函数及梯度的计算](https://zhuanlan.zhihu.com/p/21485970)       

```
def softmax_loss_naive(W, X, y, reg):
  loss = 0.0
  dW = np.zeros_like(W)
  f=np.dot(X,W)  #N*C
  fm=np.reshape(np.max(f,axis=1),(X.shape[0],1)) #N*1
  L = -np.log(np.exp(f-fm)/np.sum(np.exp(f-fm),axis=1,keepdims=True))
  for i in xrange(X.shape[0]):
    for j in xrange(W.shape[1]):
        if (y[i]==j):
            loss+=L[i,j]
            
  loss=loss/X.shape[0]
  loss+=0.5 * reg * np.sum(W * W)
  return loss, dW
```

在未经训练时，由于权重是随机生成的，因此应该每个分类的概率就是10%，因此 loss应该接近 -log (0.1) 

**计算梯度**         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter9/p2.png)

加上梯度后的softmax_loss_naive版本：

```
  num_train = X.shape[0]
  num_classes = W.shape[1]  
  f = X.dot(W)  #N by C
  f_max = np.reshape(np.max(f, axis = 1), (num_train, 1))
  prob = np.exp(f - f_max)/np.sum(np.exp(f-f_max), axis = 1, keepdims = True)
  
  for i in xrange(num_train):
    for j in xrange(num_classes):
        if (j == y[i]):
            loss += -np.log(prob[i,j])
            dW[:,j] += (1 - prob[i,j]) * X[i]
        else:
            dW[:,j] -= prob[i,j] * X[i]
            
  loss /= num_train
  loss += 0.5 * reg * np.sum(W*W)
  dW = -dW / num_train + reg * W
```


