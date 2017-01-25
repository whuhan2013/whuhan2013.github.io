---
layout: post
title: Python实现Logistic回归
date: 2017-1-25
categories: blog
tags: [python]
description: python
---

假设现在有一些数据点，我们用一条直线对这些点进行拟合(该线称为最佳拟合直线)，这个拟合过程就叫作回归。利用logistic回归进行分类的主要思想是：
根据现有数据对分类边界线建立回归公式，以此进行分类。           

**使用梯度下降找到最佳参数**        
每个回归系数初始化为1        
重复R次:      
计算整个数据集的梯度       
使用alpha*gradient更新回归系数的向量        
返回回归系数       

```
def loadDataSet():
    dataMat = []; labelMat = []
    fr = open('testSet.txt')
    for line in fr.readlines():
        lineArr = line.strip().split()
        dataMat.append([1.0, float(lineArr[0]), float(lineArr[1])])
        labelMat.append(int(lineArr[2]))
    return dataMat,labelMat

def sigmoid(inX):
    return 1.0/(1+exp(-inX))

def gradAscent(dataMatIn, classLabels):
    dataMatrix = mat(dataMatIn)             #convert to NumPy matrix
    labelMat = mat(classLabels).transpose() #convert to NumPy matrix
    m,n = shape(dataMatrix)
    alpha = 0.001
    maxCycles = 500
    weights = ones((n,1))
    for k in range(maxCycles):              #heavy on matrix operations
        h = sigmoid(dataMatrix*weights)     #matrix mult
        error = (labelMat - h)              #vector subtraction
        weights = weights + alpha * dataMatrix.transpose()* error #matrix mult
    return weights
```

**分析数据：画出决策边界**         
画出不同数据之间的分隔线         

```
def plotBestFit(weights):
    import matplotlib.pyplot as plt
    dataMat,labelMat=loadDataSet()
    dataArr = array(dataMat)
    n = shape(dataArr)[0] 
    xcord1 = []; ycord1 = []
    xcord2 = []; ycord2 = []
    for i in range(n):
        if int(labelMat[i])== 1:
            xcord1.append(dataArr[i,1]); ycord1.append(dataArr[i,2])
        else:
            xcord2.append(dataArr[i,1]); ycord2.append(dataArr[i,2])
    fig = plt.figure()
    ax = fig.add_subplot(111)
    ax.scatter(xcord1, ycord1, s=30, c='red', marker='s')
    ax.scatter(xcord2, ycord2, s=30, c='green')
    x = arange(-3.0, 3.0, 0.1)
    y = (-weights[0]-weights[1]*x)/weights[2]
    ax.plot(x, y)
    plt.xlabel('X1'); plt.ylabel('X2');
    plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter5/p1.png)

**随机梯度上升**      
梯度上升算法在每次更新回归系数时都需要遍历整个数据集,该方法在处理100个左右的数据集时尚可,但如果有数十亿样本和成千上万的特征,那么该方法的计算复杂度就太高了。   改进方法是一次仅用一个样本点来更新回归系数,该方法称为随机梯度上升算法，由于可以在新样本到来时对分类器进行增量式更新,因而随机梯度上升算法是一个在线学习算法。与 “在线学 习”相对应,一次处理所有数据被称作是“批处理” 。         

```
def stocGradAscent0(dataMatrix, classLabels):
    m,n = shape(dataMatrix)
    alpha = 0.01
    weights = ones(n)   #initialize to all ones
    for i in range(m):
        h = sigmoid(sum(dataMatrix[i]*weights))
        error = classLabels[i] - h
        weights = weights + alpha * error * dataMatrix[i]
    return weights
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter5/p2.png)


执行完毕后将得到图5-5所示的最佳拟合直线图,该图与图5-4有一些相似之处。可以看到,拟合出来的直线效果还不错,但并不像图5-4那样完美。这里的分类器错分了三分之一的样本。直接比较程序清单5-3和程序清单5-1的代码结果是不公平的,后者的结果是在整个数据集上迭代了500次才得到的。一个判断优化算法优劣的可靠方法是看它是否收敛,也就是说参数是否达到了稳定值,是否还会不断地变化?对此,我们在程序清单5-3中随机梯度上升算法上做了些修改,使其在整个数据集上运行200次。          

```
def stocGradAscent1(dataMatrix, classLabels, numIter=150):
    m,n = shape(dataMatrix)
    weights = ones(n)   #initialize to all ones
    for j in range(numIter):
        dataIndex = range(m)
        for i in range(m):
            alpha = 4/(1.0+j+i)+0.0001    #apha decreases with iteration, does not 
            randIndex = int(random.uniform(0,len(dataIndex)))#go to 0 because of the constant
            h = sigmoid(sum(dataMatrix[randIndex]*weights))
            error = classLabels[randIndex] - h
            weights = weights + alpha * error * dataMatrix[randIndex]
            del(dataIndex[randIndex])
    return weights
```

这里我们的alpha在每次迭代时都会调整，这会缓解数据波动或高频波动。同时，我们也随机选取样本来更新回归系数，这种方法也可以缓解周期性的波动。  

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter5/p3.png)
这次我们仅仅对数据集做了150次遍历,而之前的方法是500次。而达到了效果差不多的作用。       


#### 从疝气病预测马的死亡率       
本节将使用logistic回归来预测患有疝病的马的存活问题。这里的数据包含368个样本和28 个特征         

**准备数据：处理数据缺失项**        
数据中的缺失值是个非常棘手的问题,有很多文献都致力于解决这个问题。那么,数据缺失 究竟带来了什么问题?假设有100个样本和20个特征,这些数据都是机器收集回来的。若机器上 的某个传感器损坏导致一个特征无效时该怎么办?此时是否要扔掉整个数据?这种情况下,另外 19个特征怎么办?它们是否还可用?答案是肯定的。因为有时候数据相当昂贵,扔掉和重新获取 都是不可取的,所以必须采用一些方法来解决这个问题。
下面给出了一些可选的做法:     

- 使用可用特征的均值来填补缺失值;   
- 使 用 特 殊 值 来 ±真 补 缺 失 值 , 如 - 1 ;  
- 忽略有缺失值的样本;  
- 使用相似样本的均值添补缺失值;  
- 使用另外的机器学习算法预测缺失值

在我们的案例中，使用0来填补缺失值         
此外，如果类别标签已经丢失了，那么我们丢弃这个数据        

**使用logistic进行分类**           

```
def classifyVector(inX, weights):
    prob = sigmoid(sum(inX*weights))
    if prob > 0.5: return 1.0
    else: return 0.0

def colicTest():
    frTrain = open('horseColicTraining.txt'); frTest = open('horseColicTest.txt')
    trainingSet = []; trainingLabels = []
    for line in frTrain.readlines():
        currLine = line.strip().split('\t')
        lineArr =[]
        for i in range(21):
            lineArr.append(float(currLine[i]))
        trainingSet.append(lineArr)
        trainingLabels.append(float(currLine[21]))
    trainWeights = stocGradAscent1(array(trainingSet), trainingLabels, 1000)
    errorCount = 0; numTestVec = 0.0
    for line in frTest.readlines():
        numTestVec += 1.0
        currLine = line.strip().split('\t')
        lineArr =[]
        for i in range(21):
            lineArr.append(float(currLine[i]))
        if int(classifyVector(array(lineArr), trainWeights))!= int(currLine[21]):
            errorCount += 1
    errorRate = (float(errorCount)/numTestVec)
    print "the error rate of this test is: %f" % errorRate
    return errorRate

def multiTest():
    numTests = 10; errorSum=0.0
    for k in range(numTests):
        errorSum += colicTest()
    print "after %d iterations the average error rate is: %f" % (numTests, errorSum/float(numTests))
```


利用logistic回归分类，计算10次的平均错误率为33%，事实上,这个结果并不差,因 为有30%的数据缺失。当然，如果调整迭代次数，或者步长，可以进一步
降低错误率。        






