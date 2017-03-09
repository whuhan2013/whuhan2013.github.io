---
layout: post
title: 神经网络简单实现
date: 2017-3-9
categories: blog
tags: [计算机视觉]
description: 计算机视觉
---

动手实现一个简单的2维平面神经网络分类器，去分割平面上的不同类别样本点。为了循序渐进，我们打算先实现一个简单的线性分类器，然后再拓展到非线性的2层神经网络。我们可以看到简单的浅层神经网络，在这个例子上就能够有分割程度远高于线性分类器的效果。

**样本数据的产生**         
为了凸显一下神经网络强大的空间分割能力，我们打算产生出一部分对于线性分类器不那么容易分割的样本点，比如说我们生成一份螺旋状分布的样本点，如下：

```
N = 100 # 每个类中的样本点
D = 2 # 维度
K = 3 # 类别个数
X = np.zeros((N*K,D)) # 样本input
y = np.zeros(N*K, dtype='uint8') # 类别标签
for j in xrange(K):
  ix = range(N*j,N*(j+1))
  r = np.linspace(0.0,1,N) # radius
  t = np.linspace(j*4,(j+1)*4,N) + np.random.randn(N)*0.2 # theta
  X[ix] = np.c_[r*np.sin(t), r*np.cos(t)]
  y[ix] = j
# 可视化一下我们的样本点
plt.scatter(X[:, 0], X[:, 1], c=y, s=40, cmap=plt.cm.Spectral)
plt.xlim([-1,1])
plt.ylim([-1,1])
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter8/p1.png)       

紫色，红色和黄色分布代表不同的3种类别。          
一般来说，拿到数据都要做预处理，包括之前提到的去均值和方差归一化。不过我们构造的数据幅度已经在-1到1之间了，所以这里不用做这个操作。

#### 使用Softmax线性分类器         

**初始化参数**

我们先在训练集上用softmax线性分类器试试。如我们在之前的章节提到的，我们这里用的softmax分类器，使用的是一个线性的得分函数/score function，使用的损失函数是互熵损失/cross-entropy loss。包含的参数包括得分函数里面用到的权重矩阵W和偏移量b，我们先随机初始化这些参数。

```
#随机初始化参数
import numpy as np
#D=2表示维度，K=3表示类别数
W = 0.01 * np.random.randn(D,K)
b = np.zeros((1,K))
```

**计算得分**          
线性的得分函数，将原始的数据映射到得分域非常简单，只是一个直接的矩阵乘法。

```
#使用得分函数计算得分
scores = np.dot(X, W) + b
```

在我们给的这个例子中，我们有2个2维点集，所以做完乘法过后，矩阵得分scores其实是一个[300*3]的矩阵，每一行都给出对应3各类别(紫，红，黄)的得分。

**计算损失**           
然后我们就要开始使用我们的损失函数计算损失了，我们之前也提到过，损失函数计算出来的结果代表着预测结果和真实结果之间的吻合度，我们的目标是最小化这个结果。直观一点理解，我们希望对每个样本而言，对应正确类别的得分高于其他类别的得分，如果满足这个条件，那么损失函数计算的结果是一个比较低的值，如果判定的类别不是正确类别，则结果值会很高。我们之前提到了，softmax分类器里面，使用的损失函数是互熵损失。一起回忆一下，假设f是得分向量，那么我们的互熵损失是用如下的形式定义的：                 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter8/p2.png)  
好，我们实现以下，根据上面计算得到的得分scores，我们计算以下各个类别上的概率：

```
# 用指数函数还原
exp_scores = np.exp(scores)
# 归一化
probs = exp_scores / np.sum(exp_scores, axis=1, keepdims=True)
```

在我们的例子中，我们最后得到了一个[300*3]的概率矩阵prob，其中每一行都包含属于3个类别的概率。然后我们就可以计算完整的互熵损失了：

```
#计算log概率和互熵损失
corect_logprobs = -np.log(probs[range(num_examples),y])
data_loss = np.sum(corect_logprobs)/num_examples
#加上正则化项
reg_loss = 0.5*reg*np.sum(W*W)
loss = data_loss + reg_loss
```

正则化强度λ在上述代码中是reg，最开始的时候我们可能会得到loss=1.1，是通过np.log(1.0/3)得到的(假定初始的时候属于3个类别的概率一样)，我们现在想最小化损失loss

**计算梯度与梯度回传**         
我们能够用损失函数评估预测值与真实值之间的差距，下一步要做的事情自然是最小化这个值。我们用传统的梯度下降来解决这个问题。多解释一句，梯度下降的过程是：我们先选取一组随机参数作为初始值，然后计算损失函数在这组参数上的梯度(负梯度的方向表明了损失函数减小的方向)，接着我们朝着负梯度的方向迭代和更新参数，不断重复这个过程直至损失函数最小化。为了清楚一点说明这个问题，我们引入一个中间变量p，它是归一化后的概率向量，如下：
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/cs231n/chapter8/p3.png)  


