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
ax = fig.add_subplot(111)
ax.scatter(datingDataMat[:,1],datingDataMat[:2])
plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter2/p1.png)     
由于没有使用样本分类特征值，我们很难从图中看到有用的数据模式信息。        

重新输入上面的代码，调用scatter函数时使用如下代码：        
ax.scatter(datingDataMat[:,1],datingDataMat[:,2],15.0*array(datingLabels),15.0*array(datingLabels))
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter2/p2.png)     

**数据归一化**     
我们很容易发现,上面方程中数字差值最大的属性对计算结果的影响最大,也就是说,每年获取的飞行常客里程数对于计算结果的影响将远远大于表2-3中其他两个特征— 玩视频游戏的 和每周消费冰洪淋公升数— 的影响。而产生这种现象的唯一原因,仅仅是因为飞行常客里程数 远大于其他特征值。但海伦认为这三种特征是同等重要的,因此作为三个等权重的特征之一,飞 行常客里程数并不应该如此严重地影响到计算结果。


在处理这种不同取值范围的特征值时,我们通常采用的方法是将数值归一化,如将取值范围处理为0到1或者-1到1之间。下面的公式可以将任意取值范围的特征值转化为0到1区间内的值:         
newValue=(oldValue-min)/(max-min)     
其中min和max分别是数据集中的最小特征值和最大特征值。虽然改变数值取值范围增加了 分类器的复杂度,但为了得到准确结果,我们必须这样做    

```
def autoNorm(dataSet):
    minVals = dataSet.min(0)
    maxVals = dataSet.max(0)
    ranges = maxVals - minVals
    normDataSet = zeros(shape(dataSet))
    m = dataSet.shape[0]
    normDataSet = dataSet - tile(minVals, (m, 1))
    normDataSet = normDataSet / tile(ranges, (m, 1))  # element wise divide
    return normDataSet, ranges, minVals
```

**测试算法**       
前面我们巳经提到可以使用错误率来检测分类器的性能。对于分类器来说,错误率就是分类器给出错误结果的次数除以测试数据的总数,完美分类器的错误率为0,而错误率为1.0的分类器 不会给出任何正确的分类结果。代码里我们定义一个计数器变量,每次分类器错误地分类数据,计数器就加1,程序执行完成之后计数器的结果除以数据点总数即是错误率。       

```
def datingClassTest():
    hoRatio = 0.50  # hold out 10%
    datingDataMat, datingLabels = file2matrix('datingTestSet2.txt')  # load data setfrom file
    normMat, ranges, minVals = autoNorm(datingDataMat)
    m = normMat.shape[0]
    numTestVecs = int(m * hoRatio)
    errorCount = 0.0
    for i in range(numTestVecs):
        classifierResult = classify0(normMat[i, :], normMat[numTestVecs:m, :], datingLabels[numTestVecs:m], 3)
        print "the classifier came back with: %d, the real answer is: %d" % (classifierResult, datingLabels[i])
        if (classifierResult != datingLabels[i]): errorCount += 1.0
    print "the total error rate is: %f" % (errorCount / float(numTestVecs))
    print errorCount
```

分类器处理约会数据集的错误率是2.4%,这是一个相当不错的结果。依赖于分类算法、数据集和程序设置,分类器的输出结果可能有很大的不同。   
这个例子表明我们可以正确地预测分类,错误率仅仅是2.4%。海伦完全可以输人未知对象的属性信息’由分类软件来帮助她判定某一对象的可交往程度:讨厌、一般喜欢、非常喜欢。          

#### 手写识别系统       
本节我们一步步地构造使用K-近邻分类器的手写识别系统。为了简单起见,这里构造的系统只能识别数字0到9,参见图2.6。需要识别的数字已经使用图形处理软件,处理成具有相同的色彩和大小®:宽髙是32像素*32像素的黑白图像。尽管采用文本格式存储图像不能有效地利用内存空间,但是为了方便理解,我们还是将图像转换为文本格式。      

(1)收集数据:提供文本文件。            
(2)准备数据:编写函数classify0(),将图像格式转换为分类器使用的list格式。          
(3)分析数据:在python命令提示符中检查数据,确保它符合要求。       
(4)训 练 算 法 :此步驟不适用于各近邻算法。         
(5)测试算法:编写函数使用提供的部分数据集作为测试样本,测试样本与非测试样本的区别在于测试样本是已经完成分类的数据,如果预测分类与实际类别不同,则标记
为一个错误。         
(6)使用算法:本例没有完成此步驟,若你感兴趣可以构建完整的应用程序,从图像中提取数字,并完成数字识别,美国的邮件分拣系统就是一个实际运行的类似系统

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machingLearingAction/chapter2/p3.png)  
为了使用前面两个例子的分类器,我们必须将图像格式化处理为一个向量。我们将把一个32*32的二进制图像矩阵转换为1*1024的向量,这样前两节使用的分类器就可以处理数字图像信息了。       
我们首先编写一段函数img2vector将图像转换为向量:该函数创建1*1024的numpy数组,然后打开给定的文件,循环读出文件的前32行,并将每行的头32个字符值存储在numpy数 组 中,最后返回数组。

```
def img2vector(filename):
    returnVect = zeros((1, 1024))
    fr = open(filename)
    for i in range(32):
        lineStr = fr.readline()
        for j in range(32):
            returnVect[0, 32 * i + j] = int(lineStr[j])
    return returnVect
```

**测试算法**      

```
def handwritingClassTest():
    hwLabels = []
    trainingFileList = listdir('trainingDigits')  # load the training set
    m = len(trainingFileList)
    trainingMat = zeros((m, 1024))
    for i in range(m):
        fileNameStr = trainingFileList[i]
        fileStr = fileNameStr.split('.')[0]  # take off .txt
        classNumStr = int(fileStr.split('_')[0])
        hwLabels.append(classNumStr)
        trainingMat[i, :] = img2vector('trainingDigits/%s' % fileNameStr)
    testFileList = listdir('testDigits')  # iterate through the test set
    errorCount = 0.0
    mTest = len(testFileList)
    for i in range(mTest):
        fileNameStr = testFileList[i]
        fileStr = fileNameStr.split('.')[0]  # take off .txt
        classNumStr = int(fileStr.split('_')[0])
        vectorUnderTest = img2vector('testDigits/%s' % fileNameStr)
        classifierResult = classify0(vectorUnderTest, trainingMat, hwLabels, 3)
        print "the classifier came back with: %d, the real answer is: %d" % (classifierResult, classNumStr)
        if (classifierResult != classNumStr): errorCount += 1.0
    print "\nthe total number of errors is: %d" % errorCount
    print "\nthe total error rate is: %f" % (errorCount / float(mTest))
```

K-近邻算法识别手写数字数据集,错误率为1.2%       
实际使用这个算法时,算法的执行效率并不高。因为算法需要为每个测试向量做2000次距离计算,每个距离计算包括了1024个维度浮点运算,总计要执行900次,此外,我们还需要为测试向量准备2MB的存储空间。是否存在一种算法减少存储空间和计算时间的开销呢?k决策树就是K-近邻算法的优化版,可以节省大量的计算开销。

**总结**      
K-近邻算法是分类数据最简单最有效的算法,本章通过两个例子讲述了如何使用K-近邻算法构造分类器。K-近邻算法是基于实例的学习,使用算法时我们必须有接近实际数据的训练样本数 据。K-近邻算法必须保存全部数据集,如果训练数据集的很大,必须使用大量的存储空间。此外,由于必须对数据集中的每个数据计算距离值,实际使用时可能非常耗时。            


                                                          

