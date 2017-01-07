---
layout: post
title: 基于PCA和SVM的人脸识别
date: 2017-1-7
categories: blog
tags: [图像处理]
description: 基于PCA和SVM的人脸识别
---

#### svm推广到多类情况     

**一对多的最大响应策略(one against all)**    
假设有A 、B、C.. D四类样本需要划分。在抽取训练集的时候，分别按照如下4种方式划分。    

- A. 所对应的样本特征向量作为正集（类标签为+1), B、C、D所对应的样本特征向量作为负集（类标签为-1).         
- B所对应的样本特征向量作为正集，A. C. D所对应的样本特征向量作为负集       
- C所对应的样本特征向量作为正集， A. B, D所对应的样本特征向量作为负集．
- D所对应的样本特征向量作为正集， A. B、 C所对应的样本特征向量作为负集．  

对上述4个训练集分别进行训练， 得到4个SVM分类器。在测试的时候， 把未知类别的测试样本又分别送入这4个分类器进行判决,取最大值    

**一对一的投票策略(one against one 叶th voting)**      
将A. B、C, D四类样本两类两类地组成训练集， 即(A,B)、(A.C)、(A,D)、(B,C)、(B,D)、{C,D), 得到6个（对于n类问题， 为n(n-1)/2个） SVM二分器。在测试的时候，把测试样本又依次送入这6个二分类器， 采取投票形式， 最后得到一组结果。投票是以如下方式进行的。

初始化： vote(A)= vote(B)= vote(C)= vote{D)=O。        
投票过程：如果使用训练集(A,B)得到的分类器将f判定为A类，则vote(:A)=vote(A)+ 1 , 否则vote(B)=vote(B)+ 1; 如果使用(A,C)训练的分类器将又判定为A类，则vote(.4.)=vote(A)+ 1, 否则vote(C)=vote(C)+l; . ….. ; 如果使用(C,D)训练的分类器将又判定为C类，则
vote(C)=vote(C)+ 1 , 否则vote(D)=vote(D)+ 1。         
最终判决： Max(vote(A), vote(B), vote{C), vote(D))。如有两个以上的最大值， 则一般可简单地取第一个最大值所对应的类别。    

**3. 一对一的淘汰策略(one against one with. eUmlnating)**      
这是在文献中专门针对SVM提出的一种多类推广策略，实际上它也适用于所有可以提供分类器置信度信息的二分器。该方法同样基于一对一判别策略解决多类问题， 对于这4类问题， 需训练6个分类器(A,.B)、(A,C)、(A,D）、(B,C)、(B,D)、(C,D)。显然， 对于这4类中的任意一类， 如第A类中的某一样本， 就可由(A,B）、(A,C)、(A,D)这3个二分器中的任意一个来识别，即判别函数间存在冗余。于是我们将这些二分器根据其置信度从大到小排序，置信度越大表示此二分器分类的结果越可靠， 反之则越有可能出现误判。对这6个分类器按其置信度由大到小排序并分别编号， 假设为1 # (A,C), 2# (A,B）， 3#(A,D)., 4# (B,D), 5# (C,D), 6# (B,C) 。  

此时， 判别过程如下。       
(1) 设被识别对象为X, 首先由1#判别函数进行识别。若判别函数h(x) = +1, 则结果为类型A, 所有关于类型C的判别函数均被淘汰；若判别函数h(x) =-1, 则结果为类型C,所有关于类型A的判别函数均被淘汰；若判别函数h(x) = 0, 为“ 拒绝决策” 的情形， 则直接选用2#判别函数进行识别。   
(2)被识别对象x再由4#判别函数进行识别。若结果为类型+1, 淘汰所有关于D类的判别函数， 则所剩判别函数为6# (B,C)。          
(3) 被识别对象x再由6#判别函数进行识别。若得到结果为类型+1 则可判定最终的分类结果为B。  

那么， 如何来表示置信度呢？对于SVM而言，分割超平面的分类间隔越大，就说明两类样本越容易分开，表明了问题本身较好的可分性。因此可以用各个SVM二分器的分类间隔大小作为其置信度。     

#### SVM的Matlab实现      

**训练一svmtrain**       
函数svmtrain用来训练一个SVM分类器， 常用的调用语法为：      
SVMStruct = svmtrain(Tranining,Group);             
• Training是一个包含训练数据的m行n列的2维矩阵。每行表示1个训练样本（特征 向量），m表示训练样本数目； n表示样本的维数．          
• Group是一个代表训练样本类标签的1维向量．其元素值只能为0或1.通常1表示正例，0表示反例.Group的维数必须和Traningg的行数相等，以保证训练样本同其 类别标号的一一对应．              
SVMStruct是训练所得的代表SVM分类器的结构体，包含有关最佳分割超平面的种种信息，也可以计算出分类间隔值。      

除上述的常用调用形式外，还可通过＜属性名， 属性值＞形式的可选参数设置一些训练相关的高级选项， 从而实现某些自定义功能， 具体说明如下
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter11/p12.png)       

**训练结果的可视化**      
当训练数据是2维时可利用ShowPlot选项来获得训练结果的可视化解释， 调用形式如下：        
svmtrain(...,"ShowPlot",true);      

**设定错误代价C**      
在13.2.2小节讨论非线性可分情况下的C-SVM时， 介绍了错误代价系数C对于训练和分类结果的影响， 下面将给出设定C值的方法。由式(13-21)可知， 引入C对于二次规划问题求解的影响仅仅体现在约束条件当中， 因此通过在调用svmtrain时设置一个优化选项'boxconstraint'即可， 调用形式为：   
svmStruct = svmtarin(...,"boxconstraint",C);      
其中， C 为错误代价系数，默认取值为Inf, 表示错分的代价无限大， 分割超平面将倾向于尽可能最小化训练错误。通过适当地设置一个有限的C值， 将得到一个图13.7 Cc)中所示的软超平面。      

**分类-svmclassify**        
函数svmclassify的作用是利用训练得到的SVMStruct结构对一组样本进行分类，常用调用形式为：         
Group = svmclassify(SVMStruct,sample);      
SVMStruct是训练得到的代表SVM分类器的结构体， 由函数svmtrain返回．          
Sample是要进行分类的样本矩阵， 每行为1个样本特征向量，总行数等于样本数目，总列数是样本特征的维数，它必须和训练该SVM时使用的样本特征维数相同．Group是一个包含Sample中所有样本分类结果的列向量， 其维数与Sample矩阵的行数相同．    

**应用实例**      
svm训练分类鸢尾植物      

```
load fisheriris;
data = [meas(:,1),meas(:,2)];    
groups = ismember(species,'setosa');    

[train,test] = crossvalind('holdOut',groups);

svmStruct = svmtrain(data(train,:),groups(train),'showplot',true);

classes = svmclassify(svmStruct,data(test,:),'showplot',true);

nCorrect = sum(classes == groups(test));    
accuracy = nCorrect/length(classes);
```
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/dataImage/chapter11/p13.png)   




