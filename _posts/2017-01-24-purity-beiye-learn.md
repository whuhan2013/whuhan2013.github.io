---
layout: post
title: 机器学习实战之朴素贝叶斯
date: 2017-1-24
categories: blog
tags: [机器学习]
description: 机器学习
---

**基于朴素贝叶斯决策理论的分类方法**           
朴素贝叶斯是贝叶斯决策理论的一部分，所以学习朴素贝叶斯之前有必要快速了解一下贝叶斯决策理论。       
假设我们现在有一个数据集，它由两类数据组成，数据分布如图：      
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter4/p1.png)
我们用p1(x,y)表示点(x,y)属于类别1的概率，p2(x,y)表示属于类别2的概率，那么对于一个新的数据点(x,y)我们可以用下面的规则来判断它的类别 

- 如果p1(x,y)>p2(x,y),那么类别为1    
- 如果p2(x,y)>p1(x,y),那么类别为2      

也就是说,我们会选择高概率对应的类别。这就是贝叶斯决策理论的核心思想,即选择具有 最高概率的决策。                            
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter4/p2.png)

### 利用朴素贝叶斯进行文档分类    
机器学习的一个重要应用就是文档的自动分类。在文档分类中,整个文档(如一封电子邮件)是实例,而电子邮件中的某些元素则构成特征。虽然电子邮件是一种会不断增加的文本,但我们同样也可以对新闻报道、用户留言、政府公文等其他任意类型的文本进行分类。我们可以观察文档中出现的词,并把每个词的出现或者不出现作为一个特征,这样得到的特征数目就会跟词汇表中的词目一样多。朴素贝叶斯是上节介绍的贝叶斯分类器的一个扩展,是用于文档分类的常用算法。

**朴素贝叶斯的一般过程**             
(1) 收 集 数 据 :可以使用任何方法。        
(2)准备数据:需要数值型或者布尔型数据。           
(3)分析数据:有大量特征时,绘制特征作用不大,此时使用直方图效果更好。           
(4)训练算法:计算不同的独立特征的条件概率。           
(5)测试算法:计算错误率。         
(6)使用算法:一个常见的朴素贝叶斯应用是文档分类。可以在任意的分类场景中使_用朴素贝叶斯命类器,不一定非要是文本。       

朴素贝叶斯假设特征之间相互独立，并且特征同等重要，

#### 使用python进行文本分类         

**准备数据，从文本中构建词向量**             

```
def loadDataSet():
    postingList=[['my', 'dog', 'has', 'flea', 'problems', 'help', 'please'],
                 ['maybe', 'not', 'take', 'him', 'to', 'dog', 'park', 'stupid'],
                 ['my', 'dalmation', 'is', 'so', 'cute', 'I', 'love', 'him'],
                 ['stop', 'posting', 'stupid', 'worthless', 'garbage'],
                 ['mr', 'licks', 'ate', 'my', 'steak', 'how', 'to', 'stop', 'him'],
                 ['quit', 'buying', 'worthless', 'dog', 'food', 'stupid']]
    classVec = [0,1,0,1,0,1]    #1 is abusive, 0 not
    return postingList,classVec
                 
def createVocabList(dataSet):
    vocabSet = set([])  #create empty set
    for document in dataSet:
        vocabSet = vocabSet | set(document) #union of the two sets
    return list(vocabSet)

def setOfWords2Vec(vocabList, inputSet):
    returnVec = [0]*len(vocabList)
    for word in inputSet:
        if word in vocabList:
            returnVec[vocabList.index(word)] = 1
        else: print "the word: %s is not in my Vocabulary!" % word
    return returnVec
```

该函数使用词汇表或者想要检查的所有单词作为输人,然后为其中每一个单词构建一个特 征。一旦给定一篇文档(斑点犬网站上的一条留言),该文档就会被转换为词向量

**训练算法：从词向量计算概率**        

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter4/p3.png)
该函数的伪代码如下:          
计算每个类别中的文档数目         
对每篇训练文档:           
对每个类别: 如果词条出现文档中―增加该词条的计数值           
增加所有词条的计数值            
对每个类别:         
对每个词条:         
将该词条的数目除以总词条数目得到条件概率           
返回每个类别的条件概率          

