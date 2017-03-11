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
  loss+=np.sum(reg*W)
  return loss, dW
```



