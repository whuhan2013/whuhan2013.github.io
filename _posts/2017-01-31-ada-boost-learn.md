---
layout: post
title: AdaBoost算法学习
date: 2017-1-31
categories: blog
tags: [机器学习]
description: 机器学习
---

在介绍AdaBoost算法之前，需要了解一个类似的算法，装袋算法(bagging)，bagging是一种提高分类准确率的算法，通过给定组合投票的方式，获得最优解。比如你生病了，去n个医院看了n个医生，每个医生给你开了药方，最后的结果中，哪个药方的出现的次数多，那就说明这个药方就越有可能性是最优解，这个很好理解。而bagging算法就是这个思想。         

而AdaBoost算法的核心思想还是基于bagging算法，但是他又一点点的改进，上面的每个医生的投票结果都是一样的，说明地位平等，如果在这里加上一个权重，大城市的医生权重高点，小县城的医生权重低，这样通过最终计算权重和的方式，会更加的合理，这就是AdaBoost算法。AdaBoost算法是一种迭代算法，只有最终分类误差率小于阈值算法才能停止，针对同一训练集数据训练不同的分类器，我们称弱分类器，最后按照权重和的形式组合起来，构成一个组合分类器，就是一个强分类器了。算法的只要过程：            
1、对D训练集数据训练处一个分类器Ci                
2、通过分类器Ci对数据进行分类，计算此时误差率                        
3、把上步骤中的分错的数据的权重提高，分对的权重降低，以此凸显了分错的数据。         

算法流程如下：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter7/p1.png)
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter7/p2.png)

**基于单层决策树构建弱分类器**           
单层决策树是一种简单的决策树。前面我们已经介绍了决 策树的工作原理,接下来将构建一个单层决策树,而它仅基于单个特征来做决策。由于这棵树只 有一次分裂过程,因此它实际上就是一个树桩。    

单层决策树生成函数          
伪代码如下        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter7/p3.png)

```
def stumpClassify(dataMatrix,dimen,threshVal,threshIneq):#just classify the data
    retArray = ones((shape(dataMatrix)[0],1))
    if threshIneq == 'lt':
        retArray[dataMatrix[:,dimen] <= threshVal] = -1.0
    else:
        retArray[dataMatrix[:,dimen] > threshVal] = -1.0
    return retArray
    

def buildStump(dataArr,classLabels,D):
    dataMatrix = mat(dataArr); labelMat = mat(classLabels).T
    m,n = shape(dataMatrix)
    numSteps = 10.0; bestStump = {}; bestClasEst = mat(zeros((m,1)))
    minError = inf #init error sum, to +infinity
    for i in range(n):#loop over all dimensions
        rangeMin = dataMatrix[:,i].min(); rangeMax = dataMatrix[:,i].max();
        stepSize = (rangeMax-rangeMin)/numSteps
        for j in range(-1,int(numSteps)+1):#loop over all range in current dimension
            for inequal in ['lt', 'gt']: #go over less than and greater than
                threshVal = (rangeMin + float(j) * stepSize)
                predictedVals = stumpClassify(dataMatrix,i,threshVal,inequal)#call stump classify with i, j, lessThan
                errArr = mat(ones((m,1)))
                errArr[predictedVals == labelMat] = 0
                weightedError = D.T*errArr  #calc total error multiplied by D
                #print "split: dim %d, thresh %.2f, thresh ineqal: %s, the weighted error is %.3f" % (i, threshVal, inequal, weightedError)
                if weightedError < minError:
                    minError = weightedError
                    bestClasEst = predictedVals.copy()
                    bestStump['dim'] = i
                    bestStump['thresh'] = threshVal
                    bestStump['ineq'] = inequal
    return bestStump,minError,bestClasEst
```

#### 完整adaboost算法的实现        

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter7/p4.png)


