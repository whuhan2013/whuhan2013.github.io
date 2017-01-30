---
layout: post
title: Python实现SVM
date: 2017-1-26
categories: blog
tags: [python]
description: python
---

支持向量就是离分隔超平面最近的那些点，svm即最大化支持向量到分隔面的距离，找到最佳值的方法。           

**寻找最大间隔**          

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter6/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter6/p2.png)

#### SMO高效优化算法       
SMO算法是将大优化问题分解为多个小优化问题来求解的。这些小优化问题往往是很容易求解，并且对它们顺序求解的结果与将它们作为整体来求解的结果是完全一致的。          
SMO算法的目标是求出一系列的alpha和b，一旦求出了这些alpha，就很容易计算出权重向量w并得到分隔超平面。       
SMO算法的工作原理是：每次循环中选择两个alpha进行优化处理。一旦找到一对合适的alpha，那么就增大其中一个同时减少另一个。这里所谓的合适就是指两个
alpha必须要在间隔边界之外，而其第二个条件则是这两个alpha还没有进行过区间化处理或者不在边界上。      

**简化版smo算法处理小规模数据集**         
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter6/p3.png)

smo函数的伪代码如下          
创建一个alpha向量并将其初始化为0向量                      
当迭代次数小于最大迭代次数时（外循环）      
对数据集中的每个数据向量（内循环）：     
如果该数据向量可以被优化：          
随机选择另外一个数据向量        
同时优化这两个向量             
如果两个向量都不能被优化，退出内循环           
如果所有向量都没被优化，增加迭代数目，继续下一次循环       

```
def smoSimple(dataMatIn, classLabels, C, toler, maxIter):
    dataMatrix = mat(dataMatIn); labelMat = mat(classLabels).transpose()
    b = 0; m,n = shape(dataMatrix)
    alphas = mat(zeros((m,1)))
    iter = 0
    while (iter < maxIter):
        alphaPairsChanged = 0
        for i in range(m):
            fXi = float(multiply(alphas,labelMat).T*(dataMatrix*dataMatrix[i,:].T)) + b
            Ei = fXi - float(labelMat[i])#if checks if an example violates KKT conditions
            if ((labelMat[i]*Ei < -toler) and (alphas[i] < C)) or ((labelMat[i]*Ei > toler) and (alphas[i] > 0)):
                j = selectJrand(i,m)
                fXj = float(multiply(alphas,labelMat).T*(dataMatrix*dataMatrix[j,:].T)) + b
                Ej = fXj - float(labelMat[j])
                alphaIold = alphas[i].copy(); alphaJold = alphas[j].copy();
                if (labelMat[i] != labelMat[j]):
                    L = max(0, alphas[j] - alphas[i])
                    H = min(C, C + alphas[j] - alphas[i])
                else:
                    L = max(0, alphas[j] + alphas[i] - C)
                    H = min(C, alphas[j] + alphas[i])
                if L==H: print "L==H"; continue
                eta = 2.0 * dataMatrix[i,:]*dataMatrix[j,:].T - dataMatrix[i,:]*dataMatrix[i,:].T - dataMatrix[j,:]*dataMatrix[j,:].T
                if eta >= 0: print "eta>=0"; continue
                alphas[j] -= labelMat[j]*(Ei - Ej)/eta
                alphas[j] = clipAlpha(alphas[j],H,L)
                if (abs(alphas[j] - alphaJold) < 0.00001): print "j not moving enough"; continue
                alphas[i] += labelMat[j]*labelMat[i]*(alphaJold - alphas[j])#update i by the same amount as j
                                                                        #the update is in the oppostie direction
                b1 = b - Ei- labelMat[i]*(alphas[i]-alphaIold)*dataMatrix[i,:]*dataMatrix[i,:].T - labelMat[j]*(alphas[j]-alphaJold)*dataMatrix[i,:]*dataMatrix[j,:].T
                b2 = b - Ej- labelMat[i]*(alphas[i]-alphaIold)*dataMatrix[i,:]*dataMatrix[j,:].T - labelMat[j]*(alphas[j]-alphaJold)*dataMatrix[j,:]*dataMatrix[j,:].T
                if (0 < alphas[i]) and (C > alphas[i]): b = b1
                elif (0 < alphas[j]) and (C > alphas[j]): b = b2
                else: b = (b1 + b2)/2.0
                alphaPairsChanged += 1
                print "iter: %d i:%d, pairs changed %d" % (iter,i,alphaPairsChanged)
        if (alphaPairsChanged == 0): iter += 1
        else: iter = 0
        print "iteration number: %d" % iter
    return b,alphas
```

详细解释可参见：       

- [机器学习算法与Python实践之（二）支持向量机（SVM）初级](http://blog.csdn.net/zouxy09/article/details/17291543)
- [机器学习算法与Python实践之（三）支持向量机（SVM）进阶](http://blog.csdn.net/zouxy09/article/details/17291805)
- [机器学习算法与Python实践之（四）支持向量机（SVM）实现](http://blog.csdn.net/zouxy09/article/details/17292011)  

                                                          