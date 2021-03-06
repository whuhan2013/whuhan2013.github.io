---
layout: post
title: 机器学习系统设计与建议
date: 2016-12-7
categories: blog
tags: [机器学习]
description: 机器学习系统设计与建议
---

当我们运用训练好了的模型来预测未知数据的时候发现有较大的误差,我们下一步可以 做什么?         
1. 获得更多的训练实例——通常是有效的,但代价较大,下面的方法也可能有效,可 考虑先采用下面的几种方法。     
2. 尝试减少特征的数量     
3. 尝试获得更多的特征    
4. 尝试增加多项式特征     
5. 尝试减少归一化程度 λ    
6. 尝试增加归一化程度 λ       

我们不应该随机选择上面的某种方法来改进我们的算法,而是运用一些机器学习诊断法
来帮助我们知道上面哪些方法对我们的算法是有效的。     

**评估一个假设**      
当我们确定学习算法的参数的时候,我们考虑的是选择参量来使训练误差最小化,有人 认为得到一个非常小的训练误差一定是一件好事,但我们已经知道,仅仅是因为这个假设具 有很小的训练误差,并不能说明它就一定是一个好的假设函数。而且我们也学习了过拟合假 设函数的例子,所以这推广到新的训练集上是不适用的。    

因此,我们需要另一种方法来评估我们的假设函数过拟合检验。          
为了检验算法是否过拟合,我们将数据分成训练集和测试集,通常用 70%的数据作为训 练集,用剩下 30%的数据作为测试集。很重要的一点是训练集和测试集均要含有各种类型的 数据,通常我们要对数据进行“洗牌”,然后再分成训练集和测试集。    

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class6/p6.png)     

**模型选择和交叉验证集**       
假设我们要在 10 个不同次数的二项式模型之间进行选择:      
显然越高次数的多项式模型越能够适应我们的训练数据集,但是适应训练数据集并不代 表着能推广至一般情况,我们应该选择一个更能适应一般情况的模型。我们需要使用交叉验 证集来帮助选择模型。
即:使用 60%的数据作为训练集,使用 20%的数据作为交叉验证集,使用 20%的数据 作为测试集         

模型选择的方法为:     
1. 使用训练集训练出 10 个模型      
2. 用 10 个模型分别对交叉验证集计算得出交叉验证误差(代价函数的值)      
3. 选取代价函数值最小的模型        
4. 用步骤 3 中选出的模型对测试集计算得出推广误差(代价函数的值)        

**诊断偏差和方差**      
当你运行一个学习算法时,如果这个算法的表现不理想,那么多半是出现两种情况:要 么是偏差比较大,要么是方差比较大。换句话说,出现的情况要么是欠拟合,要么是过拟合 问题。那么这两种情况,哪个和偏差有关,哪个和方差有关,或者是不是和两个都有关?搞 清楚这一点非常重要,因为能判断出现的情况是这两种情况中的哪一种。其实是一个很有效 的指示器,指引着可以改进算法的最有效的方法和途径。

这个问题对于弄清如何改 进学习算法的效果非常重要,高偏差和高方差的问题基本上来说是欠拟合和过拟合的问题。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class6/p1.png) 

对于训练集,当 d 较小时,模型拟合程度更低,误差较大;随着 d 的增长,拟合程度  高,误差减小。       
对于交叉验证集,当 d 较小时,模型拟合程度低,误差较大;但是随着 d 的增长,误差呈现先减小后增大的趋势,转折点是我们的模型开始过拟合训练数据集的时候。     

如果我们的交叉验证集误差较大,我们如何判断是方差还是偏差呢? 根据上面的图表, 我们知道:      

训练集误差和交叉验证集误差近似时:偏差/欠拟合        
交叉验证集误差远大于训练集误差时:方差/过拟合      


**归一化和偏差/方差**       
在我们在训练模型的过程中,一般会使用一些归一化方法来防止过拟合。但是我们可能 会归一化的程度太高或太小了,即我们在选择 λ 的值时也需要思考与刚才选择多项式模型次 数类似的问题。     

我们选择一系列的想要测试的 λ 值,通常是 0-10 之间的呈现 2 倍关系的值(如: 0,0.01,0.02,0.04,0.08,0.15,0.32,0.64,1.28,2.56,5.12,10 共 12 个)。 我们同样把数据分为训练 集、交叉验证集和测试集。

选择 λ 的方法为:     
1. 使用训练集训练出 12 个不同程度归一化的模型      
2. 用 12 模型分别对交叉验证集计算的出交叉验证误差      
3. 选择得出交叉验证误差最小的模型       
4. 运用步骤 3 中选出模型对测试集计算得出推广误差,我们也可以同时将训练集和交叉验证集模型的代价函数误差与 λ 的值绘制在一张图表上:        

当 λ 较小时,训练集误差较小(过拟合)而交叉验证集误差较大        
随着 λ 的增加,训练集误差不断增加(欠拟合),而交叉验证集误差则是先减小后 增加     

**学习曲线**          

学习曲线就是一种很好的工具,我经常使用学习曲线来判断某一个学习算法是否处于偏 差、方差问题。学习曲线是学习算法的一个很好的合理检验(sanity check)。学习曲线是将 训练集误差和交叉验证集误差作为训练集实例数量(m)的函数绘制的图表。

即,如果我们有 100 行数据,我们从 1 行数据开始,逐渐学习更多行的数据。思想是: 当训练较少行数据的时候,训练的模型将能够非常完美地适应较少的训练数据,但是训练出 来的模型却不能很好地适应交叉验证集数据或测试集数据。

如何利用学习曲线识别高偏差/欠拟合: 作为例子,我们尝试用一条直线来适应下面的 数据,可以看出,无论训练集有多么大误差都不会有太大改观:

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class6/p2.png) 

也就是说在高偏差/欠拟合的情况下,增加数据到训练集不一定能有帮助。

如何利用学习曲线识别高方差/过拟合: 假设我们使用一个非常高次的多项式模型,并 且归一化非常小,可以看出,当交叉验证集误差远大于训练集误差时,往训练集增加更多数 据可以提高模型的效果。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class6/p3.png) 

也就是说在高方差/过拟合的情况下,增加更多数据到训练集可能可以 高算法效果。

**决定下一步做什么**       
我们已经介绍了怎样评价一个学习算法,我们讨论了模型选择问题,偏差和方差的问题。 那么这些诊断法则怎样帮助我们判断,哪些方法可能有助于改进学习算法的效果,而哪些可 能是徒劳的呢?
让我们再次回到最开始的例子,在那里寻找答案,这就是我们之前的例子。回顾 1.1 中  出的六种可选的下一步,让我们来看一看我们在什么情况下应该怎样选择:

1. 获得更多的训练实例——解决高方差    
2. 尝试减少特征的数量——解决高方差     
3. 尝试获得更多的特征——解决高偏差 
4. 尝试增加多项式特征——解决高偏差    
5. 尝试减少归一化程度 λ——解决高偏差     
6. 尝试增加归一化程度 λ——解决高方差    

神经网络的方差和偏差:

使用较小的神经网络,类似于参数较少的情况,容易导致高偏差和欠拟合,但计算代价 较小使用较大的神经网络,类似于参数较多的情况,容易导致高方差和过拟合,虽然计算代 价比较大,但是可以通过归一化手段来调整而更加适应数据。
通常选择较大的神经网络并采用归一化处理会比采用较小的神经网络效果要好。

对于神经网络中的隐藏层的层数的选择,通常从一层开始逐渐增加层数,为了更好地作选择,可以把数据分为训练集、交叉验证集和测试集,针对不同隐藏层层数的神经网络训 练神经网络, 然后选择交叉验证集代价最小的神经网络。      

#### 机器学习系统的设计     

**首先要做什么**     
以一个垃圾邮件分类器算法为例进行讨论。      
为了解决这样一个问题,我们首先要做的决定是如何选择并表达特征向量 x。我们可以 选择一个由 100 个最常出现在垃圾邮件中的词所构成的列表,根据这些词是否有在邮件中 出现,来获得我们的特征向量(出现为 1,不出现为 0),尺寸为 100×1。    
为了构建这个分类器算法,我们可以做很多事,例如:   

1. 收集更多的数据,让我们有更多的垃圾邮件和非垃圾邮件的样本     
2. 基于邮件的路由信息开发一系列复杂的特征                    
3. 基于邮件的正文信息开发一系列复杂的特征,包括考虑截词的处理        
4. 为探测刻意的拼写错误(把 watch 写成 w4tch)开发复杂的算法     

在上面这些选项中,非常难决定应该在哪一项上花费时间和精力,作出明智的选择,比随着感觉走要更好。      


**误差分析** 
构建一个学习算法的推荐方法为:                                         
1. 从一个简单的能快速实现的算法开始,实现该算法并用交叉验证集数据测试这个算 法      
2. 绘制学习曲线,决定是增加更多数据,或者添加更多特征,还是其他选择       
3. 进行误差分析:人工检查交叉验证集中我们算法中产生预测误差的实例,看看这些实例是否有某种系统化的趋势      

**类偏斜的误差度量**       
在前面的课程中,我 到了误差分析,以及设定误差度量值的重要性。那就是,设定某 个实数来评估你的学习算法,并衡量它的表现,有了算法的评估和误差度量值。有一件重要 的事情要注意,就是使用一个合适的误差度量值,这有时会对于你的学习算法造成非常微妙 的影响,这件重要的事情就是偏斜类(skewed classes)的问题。类偏斜情况表现为我们的训 练集中有非常多的同一种类的实例,只有很少或没有其他类的实例。

例如我们希望用算法来预测癌症是否是恶性的,在我们的训练集中,只有 0.5%的实例 是恶性肿瘤。假设我们编写一个非学习而来的算法,在所有情况下都预测肿瘤是良性的,那 么误差只有 0.5%。然而我们通过训练而得到的神经网络算法却有 1%的误差。这时,误差的 大小是不能视为评判算法效果的依据的。   

查准率(Precision)和查全率(Recall) 我们将算法预测的结果分成四种情况:     
1. 正确肯定(True Positive,TP):预测为真,实际为真    
2. 正确否定(True Negative,TN):预测为假,实际为假    
3. 错误肯定(False Positive,FP):预测为真,实际为假 
4. 错误否定(False Negative,FN):预测为假,实际为真      

则: 查准率=TP/(TP+FP)例,在所有我们预测有恶性肿瘤的病人中,实际上有恶性肿瘤的病
人的百分比,越高越好。 

查全率=TP/(TP+FN)例,在所有实际上有恶性肿瘤的病人中,成功预测有恶性肿瘤的
病人的百分比,越高越好。 

这样,对于我们刚才那个总是预测病人肿瘤为良性的算法,其查全率是 0。

**查全率和查准率之间的权衡**        

在之前的课程中,我们谈到查准率和召回率,作为遇到偏斜类问题的评估度量值。在很 多应用中,我们希望能够保证查准率和召回率的相对平衡。

在这节课中,我将告诉你应该怎么做,同时也向你展示一些查准率和召回率作为算法评 估度量值的更有效的方式。继续沿用刚才预测肿瘤性质的例子。假使,我们的算法输出的结 果在 0-1 之间,我们使用阀值 0.5 来预测真和假。

查准率(Precision)=TP/(TP+FP) 例,在所有我们预测有恶性肿瘤的病人中,实际上 有恶性肿瘤的病人的百分比,越高越好。     
查全率(Recall)=TP/(TP+FN)例,在所有实际上有恶性肿瘤的病人中,成功预测有恶 性肿瘤的病人的百分比,越高越好。      
如果我们希望只在非常确信的情况下预测为真(肿瘤为恶性),即我们希望更高的查准 率,我们可以使用比 0.5 更大的阀值,如 0.7,0.9。这样做我们会减少错误预测病人为恶性 肿瘤的情况,同时却会增加未能成功预测肿瘤为恶性的情况。                      
如果我们希望 高查全率,尽可能地让所有有可能是恶性肿瘤的病人都得到进一步地检 查、诊断,我们可以使用比 0.5 更小的阀值,如 0.3。   

我们可以将不同阀值情况下,查全率与查准率的关系绘制成图表,曲线的形状根据数据 的不同而不同: 

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class6/p4.png)   

**机器学习的数据** 
大部分算法,都具有相似的性能,其次,随着训练数据集的增大, 在横轴上代表以百万为单位的训练集大小,从 0.1 个百万到 1000 百万,也就是到了 10 亿规 模的训练集的样本,这些算法的性能也都对应地增强了。        
事实上,如果你选择任意一个算法,可能是选择了一个"劣等的"算法,如果你给这个劣 等算法更多的数据,那么从这些例子中看起来的话,它看上去很有可能会其他算法更好,甚 至会比"优等算法"更好。  

我觉得关键的测试:首先,一个人类专家看到了特征值 x,能很有信心的预测出 y 值吗? 因为这可以证明 y 可以根据特征值 x 被准确地预测出来。其次,我们实际上能得到一组庞 大的训练集,并且在这个训练集中训练一个有很多参数的学习算法吗?如果你能做到这两 者,那么更多时候,你会得到一个性能很好的学习算法。

**代码实现**     
[machine-learning-ex5](https://github.com/whuhan2013/IntroductionToProgrammingWithMATLAB/tree/master/machine-learning-ex5)