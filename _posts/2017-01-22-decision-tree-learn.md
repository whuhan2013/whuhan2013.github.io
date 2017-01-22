---
layout: post
title: 机器学习实战之决策树
date: 2017-1-22
categories: blog
tags: [机器学习]
description: 机器学习
---

你是否玩过二十个问题的游戏,游戏的规则很简单:参与游戏的一方在脑海里想某个事物,其他参与者向他提问题,只允许提20个问题,问题的答案也只能用对或错回答。问问题的人通过 推断分解,逐步缩小待猜测事物的范围。决策树的工作原理与20个问题类似,用户输人一系列数 据,然后给出游戏的答案。    

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter3/p1.png)

现在我们已经大致了解了决策树可以完成哪些任务,接下来我们将学习如何从一堆原始数据中构造决策树。首先我们讨论构造决策树的方法,以及如何编写构造树的python代码;接着提出 一些度量算法成功率的方法;最后使用递归建立分类器,并且使用matplotlib绘制决策树图。构造完成决策树分类器之后,我们将输人一些隐形眼镜的处方数据,并由决策树分类器预测需要的 镜片类型。    

#### 决策树的构造      
优点:计算复杂度不高,输出结果易于理解,对中间值的缺失不敏感,可以处理不相关特 征数据。           
缺 点 :可能会产生过度匹配问题。            
适用数据类型:数值型和标称型.     

在构造决策树时,我们需要解决的第一个问题就是,当前数据集上哪个特征在划分数据分类时起决定性作用。为了找到决定性的特征,划分出最好的结果,我们必须评估每个特征。完成测 试之后,原始数据集就被划分为几个数据子集。这些数据子集会分布在第一个决策点的所有分支上。如果某个分支下的数据属于同一类型，则
表示已经正确分类，如果数据子集内的数据不属于同一类型，则需要重复划分数据子集的过程。如何划分数据子集的算法与划分原始数据集的方法相同，直到所有具
有相同类型的数据均在一个数据子集内。      

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter3/p2.png)                                                                                                                   
**决策树一般流程**              
(1)收集数据:可以使用任何方法。          
(2)准备数据:树构造算法只适用于标称型数据,因此数值型数据必须离散化。        
(3)分析数据:可以使用任何方法,构造树完成之后,我们应该检查图形是否符合预期。           
(4)训练算法:构造树的数据结构。        
(5)测试算法:使用经验树计算错误率。            
(6)使用算法:此步骤可以适用于任何监督学习算法,而使用决策树可以更好地理解数据的内在含义

**信息增益**        
划分数据集的大原则是:将无序的数据变得更加有序。我们可以使用多种方法划分数据集,但是每种方法都有各自的优缺点。组织杂乱无章数据的一种方法就是使用信息论度量信息,信息 论是量化处理信息的分支科学。我们可以在划分数据之前使用信息论量化度量信息的内容。      

在划分数据集之前之后信息发生的变化称为信息增益,知道如何计算信息增益,我们就可以计算每个特征值划分数据集获得的信息增益,获得信息增益最高的特征就是最好的选择。                

在可以评测哪种数据划分方式是最好的数据划分之前,我们必须学习如何计算信息增益。集合信息的度量方式称为香农熵或者简称为熵,这个名字来源于信息论之父克劳德•香农。       
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter3/p3.png)

下面我们用python计算信息熵,此代码的功能是计算给定数据集的熵

```
def calcShannonEnt(dataSet):
    numEntries = len(dataSet)
    labelCounts = {}
    for featVec in dataSet:  # the the number of unique elements and their occurance
        currentLabel = featVec[-1]
        if currentLabel not in labelCounts.keys(): labelCounts[currentLabel] = 0
        labelCounts[currentLabel] += 1
    shannonEnt = 0.0
    for key in labelCounts:
        prob = float(labelCounts[key]) / numEntries
        shannonEnt -= prob * log(prob, 2)  # log base 2
    return shannonEnt
```

得到熵之后,我们就可以按照获取最大信息增益的方法划分数据集,下一节我们将具体学习 如何划分数据集以及如何度量信息增益。       

**划分数据集**          
分类算法除了需要测量信息熵,还需要划分数 据集,度量花费数据集的熵,以便判断当前是否正确地划分了数据集。我们将对每个特征划分数据集的结果计算一次信息熵,然后判断按照哪个特征划分数据集是最好的划分方式。想象一个分布在二维空间的数据散点图,需要在数据之间划条线,将它们分成两部分,我们应该按照x轴还 是y轴划线呢?          

```
def splitDataSet(dataSet, axis, value):
    retDataSet = []
    for featVec in dataSet:
        if featVec[axis] == value:
            reducedFeatVec = featVec[:axis]  # chop out axis used for splitting
            reducedFeatVec.extend(featVec[axis + 1:])
            retDataSet.append(reducedFeatVec)
    return retDataSet
```

接下来我们遍历整个数据集，循环计算香农熵和splitDataSet()函数，找到最好的特征划分方式。熵计算会告诉我们如何划分数据集是最好的数据组织方式。     
```
def chooseBestFeatureToSplit(dataSet):
    numFeatures = len(dataSet[0]) - 1  # the last column is used for the labels
    baseEntropy = calcShannonEnt(dataSet)
    bestInfoGain = 0.0;
    bestFeature = -1
    for i in range(numFeatures):  # iterate over all the features
        featList = [example[i] for example in dataSet]  # create a list of all the examples of this feature
        uniqueVals = set(featList)  # get a set of unique values
        newEntropy = 0.0
        for value in uniqueVals:
            subDataSet = splitDataSet(dataSet, i, value)
            prob = len(subDataSet) / float(len(dataSet))
            newEntropy += prob * calcShannonEnt(subDataSet)
        infoGain = baseEntropy - newEntropy  # calculate the info gain; ie reduction in entropy
        if (infoGain > bestInfoGain):  # compare this to the best gain so far
            bestInfoGain = infoGain  # if better than current best, set to best
            bestFeature = i
    return bestFeature  # returns an integer
```

遍历当前特征中的所有唯一属性值,对每个特征划分一次数据集,然后计算数据集的新熵值,并对所有唯一特征值得到的熵求和。信息增益是熵的减少或者是数据无序度的减少,大家肯 定对于将熵用于度量数据无序度的减少更容易理解。最后,比较所有特征中的信息增益,返回最 好特征划分的索引值

**递归构建决策树**              
目前我们已经学习了从数据集构造决策树算法所需要的子功能模块,其工作原理如下:得到原始数据集，然后基于最好的属性值划分数据集，由于特征值可能多于两个，因此可能存在大于两个分支的数据集划分。第一次划分后，数据将被向下传递到树分支的下一个节点，在这个节点上，我们可以再次划分数据，因此我们可以采用递归的原则处理数据集。                
递归结束的条件是：程序遍历完所有划分数据集的属性，或者每个分支下的所有实例都具有相同的分类。如果所有实例具有相同的分类，则得到一个叶子节点或者终止块。任何到达叶子结点的数据必然属于叶子节点的分类。        
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter3/p4.png)                                                                                                                  
如果数据集已经处理了所有属性，但是类标签依然不是唯一的，此时我们需要决定如何定义该叶子节点，在这种情况下，我们通常采用多数表决的方法来决定该叶子节点的分类。     

```
def majorityCnt(classList):
    classCount = {}
    for vote in classList:
        if vote not in classCount.keys(): classCount[vote] = 0
        classCount[vote] += 1
    sortedClassCount = sorted(classCount.iteritems(), key=operator.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]
```

接下来创建树的代码     

```
def createTree(dataSet, labels):
    classList = [example[-1] for example in dataSet]
    if classList.count(classList[0]) == len(classList):
        return classList[0]  # stop splitting when all of the classes are equal
    if len(dataSet[0]) == 1:  # stop splitting when there are no more features in dataSet
        return majorityCnt(classList)
    bestFeat = chooseBestFeatureToSplit(dataSet)
    bestFeatLabel = labels[bestFeat]
    myTree = {bestFeatLabel: {}}
    del (labels[bestFeat])
    featValues = [example[bestFeat] for example in dataSet]
    uniqueVals = set(featValues)
    for value in uniqueVals:
        subLabels = labels[:]  # copy all of labels, so trees don't mess up existing labels
        myTree[bestFeatLabel][value] = createTree(splitDataSet(dataSet, bestFeat, value), subLabels)
    return myTree
```

变量卿myTree包含了很多代表树结构信息的嵌套字典,从左边开始,第一个关键字nosurfaCing是第一个划分数据集的特征名称,该关键字的值也是另一个数据字典。第二个关键字 是nosurfaCinf特征划分的数据集,这些关键字的值是nosurfaCing节点的子节点。这些值可能是类标签,也可能是另一个数据字典。如果值是类标签,则该子节点是叶子节点;如果值是另一个数据字典,则子节点是一个判断节点,这种格式结构不断重复就构成了整棵树。本节的例子中,这棵树包含了3个叶子节点以及2个判断节点.      

#### python中使用matplotlib绘制树      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter3/p5.png)
                                                                                                
**matplotlib注解**       
matplotlib提供了一个注解工具annotion,非常有用,它可以在数据图形上添加文本注释。注解通常用于解释数据的内容。由于数据上面直接存在文本描述非常丑陋,因此工具内嵌支 持带箭头的划线工具,使得我们可以在其他恰当的地方指向数据位置,并在此处添加描述信息, 解 释 数 据 内 容.

**使用文本注解绘制树结点**           

```
decisionNode = dict(boxstyle="sawtooth", fc="0.8")
leafNode = dict(boxstyle="round4", fc="0.8")
arrow_args = dict(arrowstyle="<-")

def plotNode(nodeTxt, centerPt, parentPt, nodeType):
    createPlot.ax1.annotate(nodeTxt, xy=parentPt, xycoords='axes fraction',
                            xytext=centerPt, textcoords='axes fraction',
                            va="center", ha="center", bbox=nodeType, arrowprops=arrow_args)

def createPlot():
   fig = plt.figure(1, facecolor='white')
   fig.clf()
   createPlot.ax1 = plt.subplot(111, frameon=False) #ticks for demo puropses
   plotNode('a decision node', (0.5, 0.1), (0.1, 0.5), decisionNode)
   plotNode('a leaf node', (0.8, 0.1), (0.3, 0.8), leafNode)
   plt.show()
```

**构造注解树**          
绘制一棵完整的树需要一些技巧，需要了解有多少叶结点，以确定x轴的长度，同时要知道共有多少层，以便可以确定y轴的高度。     

获取叶结点的数目与树的层次         

```
def getNumLeafs(myTree):
    numLeafs = 0
    firstStr = myTree.keys()[0]
    secondDict = myTree[firstStr]
    for key in secondDict.keys():
        if type(secondDict[
                    key]).__name__ == 'dict':  # test to see if the nodes are dictonaires, if not they are leaf nodes
            numLeafs += getNumLeafs(secondDict[key])
        else:
            numLeafs += 1
    return numLeafs


def getTreeDepth(myTree):
    maxDepth = 0
    firstStr = myTree.keys()[0]
    secondDict = myTree[firstStr]
    for key in secondDict.keys():
        if type(secondDict[
                    key]).__name__ == 'dict':  # test to see if the nodes are dictonaires, if not they are leaf nodes
            thisDepth = 1 + getTreeDepth(secondDict[key])
        else:
            thisDepth = 1
        if thisDepth > maxDepth: maxDepth = thisDepth
    return maxDepth
```

开始绘制树      

```
def plotTree(myTree, parentPt, nodeTxt):  # if the first key tells you what feat was split on
    numLeafs = getNumLeafs(myTree)  # this determines the x width of this tree
    depth = getTreeDepth(myTree)
    firstStr = myTree.keys()[0]  # the text label for this node should be this
    cntrPt = (plotTree.xOff + (1.0 + float(numLeafs)) / 2.0 / plotTree.totalW, plotTree.yOff)
    plotMidText(cntrPt, parentPt, nodeTxt)
    plotNode(firstStr, cntrPt, parentPt, decisionNode)
    secondDict = myTree[firstStr]
    plotTree.yOff = plotTree.yOff - 1.0 / plotTree.totalD
    for key in secondDict.keys():
        if type(secondDict[
                    key]).__name__ == 'dict':  # test to see if the nodes are dictonaires, if not they are leaf nodes
            plotTree(secondDict[key], cntrPt, str(key))  # recursion
        else:  # it's a leaf node print the leaf node
            plotTree.xOff = plotTree.xOff + 1.0 / plotTree.totalW
            plotNode(secondDict[key], (plotTree.xOff, plotTree.yOff), cntrPt, leafNode)
            plotMidText((plotTree.xOff, plotTree.yOff), cntrPt, str(key))
    plotTree.yOff = plotTree.yOff + 1.0 / plotTree.totalD


# if you do get a dictonary you know it's a tree, and the first element will be another dict

def createPlot(inTree):
    fig = plt.figure(1, facecolor='white')
    fig.clf()
    axprops = dict(xticks=[], yticks=[])
    createPlot.ax1 = plt.subplot(111, frameon=False, **axprops)  # no ticks
    # createPlot.ax1 = plt.subplot(111, frameon=False) #ticks for demo puropses
    plotTree.totalW = float(getNumLeafs(inTree))
    plotTree.totalD = float(getTreeDepth(inTree))
    plotTree.xOff = -0.5 / plotTree.totalW;
    plotTree.yOff = 1.0;
    plotTree(inTree, (0.5, 1.0), '')
    plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter3/p6.png)







                                                                                                                                                                                                                                                                                    
                                                                                                                                            
