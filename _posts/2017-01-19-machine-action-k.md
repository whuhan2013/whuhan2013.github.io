---
layout: post
title: 机器学习实战之K近邻算法
date: 2017-1-19
categories: blog
tags: [机器学习]
description: 机器学习
---


**k近邻算法概述**            
简单地说,K近邻算法采用测量不同特征值之间的距离方法进行分类。      

优 点 :精度高、对异常值不敏感、无数据输入假定。        
缺点:计算复杂度高、空间复杂度高。          
适用数据范围:数值型和标称型。

它的工作原理是:存在一个样本数 据集合,也称作训练样本集,并且样本集中每个数据都存在标签,即我们知道样本集中每一数据 与所属分类的对应关系。输人没有标签的新数据后,将新数据的每个特征与样本集中数据对应的 特征进行比较,然后算法提取样本集中特征最相似数据(最近邻)的分类标签。一般来说,我们 只选择样本数据集中前K个最相似的数据,这就是K近邻算法中K的出处,通常K是不大于20的整数。 最后,选择K个最相似数据中出现次数最多的分类,作为新数据的分类。         

**K近邻算法的一般流程**                         
(1)收集数据:可以使用任何方法。            
(2)准备数据:距离计算所需要的数值,最好是结构化的数据格式。          
(3)分析数据:可以使用任何方法。           
(4)训练算法:此步驟不适用于1 近邻算法。             
(5)测试算法:计算错误率。             
(6)使用算法:首先需要输入样本数据和结构化的输出结果,然后运行k-近邻算法判定输入数据分别属于哪个分类,最后应用对计算出的分类执行后续的处理。  

**K近邻算法伪代码实现**       
对未知类别属性的数据集中的每个点依次执行以下操作:            
(1)计算已知类别数据集中的点与当前点之间的距离,常常使用欧氏距离公式;                        
(2)按照距离递增次序排序;         
(3)选取与当前点距离最小的K个点;             
(4)确定前K个点所在类别的出现频率;             
(5)返回前K个点出现频率最高的类别作为当前点的预测分类。       

```
def classify0(inX, dataSet, labels, k):
    dataSetSize = dataSet.shape[0]
    diffMat = tile(inX, (dataSetSize, 1)) - dataSet
    sqDiffMat = diffMat ** 2
    sqDistances = sqDiffMat.sum(axis=1)
    distances = sqDistances ** 0.5
    sortedDistIndicies = distances.argsort()
    classCount = {}
    for i in range(k):
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel, 0) + 1
    sortedClassCount = sorted(classCount.iteritems(), key=operator.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]
```

####  使用k近邻算法改进约会网站的配对效果        
(1)收集数据:提供文本文件。
(2)准备数据: 使用python解析文本文件。           
(3)使用Matplotlib画二维扩散图        
(4)训练算法:此步驟不适用于k近邻算法。                
(5)测试算法:使用海伦提供的部分数据作为测试样本。         
测试样本和非测试样本的区别在于:测试样本是已经完成分类的数据,如果预测分类与实际类别不同,则标记为一个错误。            
(6)使用算法:产生简单的命令行程序,然后海伦可以输入一些特征数据以判断对方是否为自己喜欢的类型。         

**从文本文件中解析数据**        
海伦收集约会数据巳经有了一段时间,她把这些数据存放在文本文件中 ,每 个样本数据占据一行,总共有1000行。海伦的样本主要包含以下3种特征:        
□ 每年获得的飞行常客里程数        
□ 玩视频游戏所耗时间百分比         
□ 每周消费的冰淇淋公升数       

我们通过file2matrix函数读入数据,该函数的输人为文 件名字符串 输出为训练样本矩阵和类标签向量。           

```
def file2matrix(filename):
    fr = open(filename)
    numberOfLines = len(fr.readlines())  # get the number of lines in the file
    returnMat = zeros((numberOfLines, 3))  # prepare matrix to return
    classLabelVector = []  # prepare labels return
    fr = open(filename)
    index = 0
    for line in fr.readlines():
        line = line.strip()
        listFromLine = line.split('\t')
        returnMat[index, :] = listFromLine[0:3]
        classLabelVector.append(int(listFromLine[-1]))
        index += 1
    return returnMat, classLabelVector
```  

**分析数据：使用matplotlib创建散点图**         

在python命令环境下，输入下列命令     

```
import matplotlib
import matplotlib.pyplot as plt
fig = plt.figure()
ax = fig.addsubplot(111)
ax.scatter(datingDataMat[:,1],datingDataMat[:2])
plot.show()
```



