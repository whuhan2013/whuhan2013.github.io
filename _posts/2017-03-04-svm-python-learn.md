---
layout: post
title: SVM分类器python实现
date: 2017-3-4
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---


本作业的目标如下：           

- implement a fully-vectorized loss function for the SVM
- implement the fully-vectorized expression for its analytic gradient
- check your implementation using numerical gradient
- use a validation set to tune the learning rate and regularization strength
- optimize the loss function with SGD
- visualize the final learned weights

**归一化图片**       

```
mean_image = np.mean(X_train, axis=0)
print mean_image[:10] # print a few of the elements
plt.figure(figsize=(4,4))
plt.imshow(mean_image.reshape((32,32,3)).astype('uint8')) # visualize the mean image
plt.show()
X_train -= mean_image
X_val -= mean_image
X_test -= mean_image
X_dev -= mean_image
```

**损失函数与梯度计算**           

```
def svm_loss_naive(W, X, y, reg):
  dW = np.zeros(W.shape) # initialize the gradient as zero

  # compute the loss and the gradient
  num_classes = W.shape[1]
  num_train = X.shape[0]
  loss = 0.0
  for i in xrange(num_train):
    scores = X[i].dot(W)
    correct_class_score = scores[y[i]]
    for j in xrange(num_classes):
      if j == y[i]:
        continue
      margin = scores[j] - correct_class_score + 1 # note delta = 1
      
      if margin > 0:
        loss += margin
        dW[:, y[i]] += -X[i, :]     # compute the correct_class gradients
        dW[:, j] += X[i, :]         # compute the wrong_class gradients
  # Right now the loss is a sum over all training examples, but we want it
  # to be an average instead so we divide by num_train.
  loss /= num_train
  dW /= num_train
  dW += 2 * reg * W 

  # Add regularization to the loss.
  loss += 0.5 * reg * np.sum(W * W)
  
  return loss, dW
```

关于梯度的计算：详情可参见[SVM损失函数及梯度矩阵的计算](https://zhuanlan.zhihu.com/p/21478575?refer=baina)


