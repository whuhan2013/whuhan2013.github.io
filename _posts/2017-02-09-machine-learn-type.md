---
layout: post
title: 机器学习分类问题
date: 2017-2-9
categories: blog
tags: [机器学习基石与技法]
description: 机器学习基石与技法
---

机器学习方法的分类学，通过不同的分类标准来讨论。         

#### 根据输出空间来分类。

1， 分类(classification)        
1.1 二值分类 (binary classification)：输出为 {+1， -1}。         
1.2 多值分类 (multiclass classification)：输出为有限个类别，{1, 2, 3, ... , K}         

2， 回归(regression)                       
输出空间：实数集 R ， 或 区间 [lower, upper]                

3， 结构学习（structured learning）：典型的有序列化标注问题                         
输出是一个结构（如句子中每个单词的词性）， 可以成为 hyperclass， 通常难以显示地定义该类。             

需要重点研究的是二值分类和回归     

#### 根据数据标签(label) 情况来分类。

1， 有监督学习(supervised learning)：训练数据中每个xi 对应一个标签yi。         
应用：分类

2， 无监督学习(unsupervised learning)：没有指明每个xi 对应的是什么，即对x没有label。         
应用：聚类，密度估计(density estimation)， 异常检测。       

3， 半监督学习(semi-supervised learning)：只有少量标注数据，利用未标注数据。            
应用：人脸识别；医药效果检测。                     

4， 增强学习(reinforcement learning)                        ：通过隐含信息学习，通常无法直接表示什么是正确的，但是可以通过”惩罚“不好的结果，”奖励“好的结果来优化学习效果。        
应用：广告系统，扑克、棋类游戏。     

总结：有监督学习有所有的yi；无监督学习没有yi；半监督学习有少量的yi；增强学习有隐式的yi。       

#### 根据不同的协议来分类。

1， 批量学习 - Batch learning        
利用所有已知训练数据来学习。            

2， 在线学习 - online learning         
通过序列化地接收数据来学习，逐渐提高性能。           
应用：垃圾邮件， 增强学习。         

3， 主动学习 - active learning          
learning by asking：开始只有少量label, 通过有策略地”问问题“ 来提高性能。             
比如遇到xi, 不确定输出是否正确，则主动去确认yi 是什么，依次来提高性能。         

#### 通过输入空间来分类。

1， 离散特征 - concrete features           
特征中通常包含了人类的智慧。例如对硬币分类需要的特征是（大小，重量）；对信用分级需要的特征是客户的基本信息。这些特征中已经蕴含了人的思考。

2， 原始特征 - raw features                              
这些特征对于学习算法来说更加困难，通常需要人或机器（深度学习，deep learning）将这些特征转化为离散(concrete)特征。     
例如，数字识别中，原始特征是图片的像素矩阵；声音识别中的声波信号；机器翻译中的每个单词。         

3， 抽象特征 - abstract features                         
抽象特征通常没有任何真实意义，更需要认为地进行特征转化、抽取和再组织。                  
例如，预测某用户对电影的评分，原始数据是(userid, itemid, rating)， rating 是训练数据的标签，相当于y。这里的(userid, itemid)本身对学习任务是没有任何帮助的，我们必须对数据所进一步处理、提炼、再组织。

总结：离散特征具有丰富的自然含义；原始特征有简单的自然含义；抽象特征没有自然含义。                      
原始特征、抽象特征都需要再处理，此过程成为特征工程(feature engineering)，是机器学习、数据挖掘中及其重要的一步。离散特征一般只需要简单选取就够了。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/foundation/chapter3/p1.jpg)
