---
layout: post
title: 支持向量机学习
date: 2016-12-10
categories: blog
tags: [机器学习]
description: 支持向量机
---

与逻辑回归和神经网络相比,支持向量机,或者简称 SVM,在学习复杂的非线性 方程时 供了一种更为清晰,更加强大的方式      

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p1.png) 

如果我们用一个新的代价函数来代替,即这条从 0 点开始的水平直线,然后是一条斜 线,像上图。那么,现在让我给这两个方程命名,左边的函数,我称之为$cos t_1(z)$ ,同时,
右边函数我称它为$cost_0(z)$。这里的下标是指在代价函数中,对应的 y=1 和 y=0 的情况, 拥有了这些定义后,现在,我们就开始构建支持向量机。  

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p3.png) 

这是我们在逻辑回归中使用代价函数 J(θ)。也许这个方程看起来不是非常熟悉。这是因 为之前有个负号在方程外面,但是,这里我所做的是,将负号移到了表达式的里面,这样做
使得方程看起来有些不同,为实现svm，我们对其进行了一系列变换。

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p2.png) 

最后有别于逻辑回归输出的概率。在这里,我们的代价函数,当最小化代价函数,获得 参数 θ 时,支持向量机所做的是它来直接预测 y 的值等于 1,还是等于 0。因此,这个假设
函数会预测 1,当  $\theta^Tx$ 大于或者等于 0 时.所以学习参数 θ 就是支持向量机
假设函数的形式。那么,这就是支持向量机数学上的定义。

**大边界的直观理解**       
人们有时将支持向量机看作是大间距分类器。在这一部分,我将介绍其中的含义,这有 助于我们直观理解 SVM 模型的假设是什么样的。     

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p4.png) 

这是支持向量机的一个有趣性质。事实上,如果你有一个正样本 y 等
于 1,则其实我们仅仅要求$\theta^Tx$ 大于等于 0,就能将该样本恰当分出,这是因为如果$\theta^Tx$>0大的话,我们的模型代价函数值为 0,类似地,如果你有一个负样本,则仅需要$\theta^Tx$>0就
会将负例正确分离,但是,支持向量机的要求更高,不仅仅要能正确分开输入的样本,即不 仅仅要求$\theta^Tx$>0,我们需要的是比 0 值大很多,比如大于等于 1,我也想这个比 0 小很多,
比如我希望它小于等于-1,这就相当于在支持向量机中嵌入了一个额外的安全因子。或者说 安全的间距因子。    

当然,逻辑回归做了类似的事情。但是让我们看一下,在支持向量机中,这个因子会导 致什么结果。具体而言,我接下来会考虑一个特例。我们将这个常数 C 设置成一个非常大的值。比如我们假设 C 的值为 100000 或者其它非常大的数,然后来观察支持向量机会给出什么结果?

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p5.png) 

如果 C 非常大,则最小化代价函数的时候,我们将会很希望找到一个使第一项为 0 的 最优解。因此,让我们尝试在代价项的第一项为 0 的情形下理解该优化问题。比如我们可以 把 C 设置成了非常大的常数,这将给我们一些关于支持向量机模型的直观感受

这将遵从以下的约束: $\theta^Tx^{(i)}>=1$,如果y(i)是等于1的, $\theta^Tx^{(i)}<=-1$,如果样本i是 一个负样本,这样当你求解这个优化问题的时候,当你最小化这个关于变量 θ 的函数的时
候,你会得到一个非常有趣的决策边界。      

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p6.png) 
具体而言,如果你考察这样一个数据集,其中有正样本,也有负样本,可以看到这个数 据集是线性可分的。我的意思是,存在一条直线把正负样本分开。当然有多条不同的直线, 可以把正样本和负样本完全分开。      

比如,这就是一个决策边界可以把正样本和负样本分开。但是多多少少这个看起来并不 是非常自然是么?    

或者我们可以画一条更差的决策界,这是另一条决策边界,可以将正样本和负样本分开, 但仅仅是勉强分开,这些决策边界看起来都不是特别好的选择,支持向量机将会选择这个黑 色的决策边界,相较于之前我用粉色或者绿色画的决策界。这条黑色的看起来好得多,黑线 看起来是更稳健的决策界。在分离正样本和负样本上它显得的更好。数学上来讲,这是什么 意思呢?这条黑线有更大的距离,这个距离叫做间距 (margin)。
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p7.png) 

当画出这两条额外的蓝线,我们看到黑色的决策界和训练样本之间有更大的最短距离。 然而粉线和蓝线离训练样本就非常近,在分离样本的时候就会比黑线表现差。因此,这个距 离叫做支持向量机的间距,而这是支持向量机具有鲁棒性的原因,因为它努力用一个最大间 距来分离样本。因此支持向量机有时被称为大间距分类器,而这其实是求解上一页幻灯片上 优化问题的结果。

在本节课中关于大间距分类器,我想讲最后一点:我们将这个大间距分类器中的正则化 因子常数 C 设置的非常大,我记得我将其设置为了 100000,因此对这样的一个数据集,也 许我们将选择这样的决策界,从而最大间距地分离开正样本和负样本。那么在让代价函数最
小化的过程中,我们希望找出在 y=1 和 y=0 两种情况下都使得代价函数中左边的这一项尽量为零的参数。如果我们找到了这 样的参数,则我们的最小化问题便转变成:     
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p8.png) 

回顾 C=1/λ,因此:      
C 较大时,相当于 λ 较小,可能会导致过拟合,高方差。       
C 较小时,相当于 λ 较大,可能会导致低拟合,高偏差。

**数学背后的大边界分类**      
根据$u^Tv=p●||u||$ ,    
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p9.png) 
因此支持向量机做的全部事情,就是极小化参数向量 θ 范数的平方,或者说长度 的平方,即使p变大，则可以取得最小值。      


#### 核函数       

回顾我们之前讨论过可以使用高级数的多项式模型来解决无法用直线进行分隔的分类问题，我们可以通过创建多项式模型来解决问题
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p10.png) 

这些地标的作用是什么?如果一个训练实例 x 与地标 L 之间的距离近似于 0,则新特征
f 近似于 $e^{-0}=1$,如果训练实例 x 与地标 L 之间距离较远,则 f 近似于 $e^{-(一个较大的数)}=0$。 假设我们的训练实例含有两个特征[x1 x2],给定地标$l^{(1)}$与不同的σ值,见下图:
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p11.png) 
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p12.png) 

**如何选择地标?**           
我们通常是根据训练集的数量选择地标的数量,即如果训练集中有 m 个实例,则我们 选取 m 个地标,并且令:$l^{(1)}=x^{(1)},l^{(2)}=x^{(2)},...,l^{(m)}=x^{(m)}$。这样做的好处在于:现在我们得到的新特 征是建立在原有特征与训练集中所有其他特征之间距离的基础之上的,即

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p13.png) 

下面我们将核函数运用到支持向量机中,修改我们的支持向量机假设为:      
• 给定 x,计算新特征 f,当 $θ^Tf>=0$ 时,预测 y=1,否则反之。 相应地修改代价函数 为:
![](https://raw.githubusercontent.com/whuhan2013/myImage/master/machineLearning/class7/p14.png) 

在此,我们不介绍最小化支持向量机的代价函数的方法,你可以使用现有的软件包(如 liblinear,libsvm 等)。在使用这些软件包最小化我们的代价函数之前,我们通常需要编写核 函数,并且如果我们使用高斯核函数,那么在使用之前进行特征缩放是非常必要的。

另外,支持向量机也可以不使用核函数,不使用核函数又称为线性核函数(linear kernel), 当我们不采用非常复杂的函数,或者我们的训练集特征非常多而实例非常少的时候,可以采 用这种不带核函数的支持向量机。

下面是支持向量机的两个参数 C 和 σ 的影响:     
C 较大时,相当于 λ 较小,可能会导致过拟合,高方差;    
C 较小时,相当于 λ 较大,可能会导致低拟合,高偏差;    
σ 较大时,导致高方差;    
σ 较小时,导致高偏差。     

#### 使用支持向量机      
目前为止,我们已经讨论了 SVM 比较抽象的层面,在这个视频中我将要讨论到为了运
行或者运用 SVM。不建议你自己写软件来求解参数 θ,有许多好的软件库,我正好用得最多的两个是 liblinear 和 libsvm,但是真的有很 多软件库可以用来做这件事儿     

在高斯核函数之外我们还有其他一些选择,如:    
多项式核函数(Polynomial Kernel)   
字符串核函数(String kernel)    
卡方核函数( chi-square kernel)                
直方图交集核函数(histogram intersection kernel)       
等等... 这些核函数的目标也都是根据训练集和地标之间的距离来构建新特征,这些核函数需要
满足 Mercer's 定理,才能被支持向量机的优化软件正确处理     

**多类分类问题**      
假设我们利用之前介绍的一对多方法来解决一个多类分类问题。如果一共有 k 个类,则 我们需要 k 个模型,以及 k 个参数向量 θ。我们同样也可以训练 k 个支持向量机来解决多类 分类问题。但是大多数支持向量机软件包都有内置的多类分类功能,我们只要直接使用即可。

尽管你不去写你自己的 SVM(支持向量机)的优化软件,但是你也需要做几件事:     
1、是 出参数 C 的选择。我们在之前的视频中讨论过误差/方差在这方面的性质。          

2、你也需要选择内核参数或你想要使用的相似函数,其中一个选择是:我们选择不需
要任何内核参数,没有内核参数的理念,也叫线性核函数。因此,如果有人说他使用了线性
核的 SVM(支持向量机),这就意味这他使用了不带有核函数的 SVM(支持向量机)     

从逻辑回归模型,我们得到了支持向量机模型,在两者之间,我们应该如何选择呢?  

下面是一些普遍使用的准则:     
n 为特征数,m 为训练样本数。     

(1)如果相较于 m 而言,n 要大许多,即训练集数据量不够支持我们训练一个复杂的非
线性模型,我们选用逻辑回归模型或者不带核函数的支持向量机。      
(2)如果 n 较小,而且 m 大小中等,例如 n 在 1-1000 之间,而 m 在 10-10000 之间,
使用高斯核函数的支持向量机。            
(3)如果 n 较小,而 m 较大,例如 n 在 1-1000 之间,而 m 大于 50000,则使用支持向量
机会非常慢,解决方案是创造、增加更多的特征,然后使用逻辑回归或不带核函数的支持向 量机。         












