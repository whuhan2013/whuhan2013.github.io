---
layout: post
title: 机器学习资源
date: 2016-6-26
categories: blog
tags: [收藏]
description: 机器学习资源
---   


### 转载 

作者：林洲汉
链接：https://www.zhihu.com/question/44864396/answer/106563622      
来源：知乎              
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。           

**阶段一：初学乍练** 

起步的东西当然要实在一点好玩一点的才好，不要教科书那些不知所云的公式。作为一个python派，怎能不推荐用Python＋Theano的组合先跑跑实际的模型。

首先，花一天时间看完python的基本语法。网上有一本很简洁易用的“速成宝典”《[简明 Python 教程](https://link.zhihu.com/?target=http%3A//old.sebug.net/paper/python/)》。看一遍真的只要一天！然后大概看一下numpy，基本就和matlab差不多。接下去就是 [Deep Learning Tutorials](https://link.zhihu.com/?target=http%3A//deeplearning.net/tutorial/)。把这个网页里的前几个模型看一遍，相应的代码下载下来跑通。后面有复杂的看不懂不要紧，先放一边。至少这个时候你有了一个手写字符识别的机器学习系统了。而且对模型、数据、算法这几个部分有了一个大体的了解。知道所谓的机器学习一般是怎么做的了。

好了现在你已经可以吹牛说你会最基本的人工智能模型了。当然吹牛归吹牛，吹完了赶紧回来看书。


**阶段二：初窥门径**   

要做到真的会机器学习，还得对机器学习的算法有一个比较全面的了解。过完阶段二，你才可以不吹牛地说你会最基本的人工智能了。

推荐几本书跟一个在线课程：   

- Andrew Ng的coursera 招牌菜《machine learning》。基础不太好的同学可以看这门课的[coursera](https://link.zhihu.com/?target=https%3A//www.coursera.org/learn/machine-learning)版，基础好的请移步这门课的[youtube](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DUzxYlbK2c7E)版本。Andrew在斯坦福教的版本（即youtube版，CS229）比coursera上的难多了。虽然自学不会有人来批作业，可是课程的assignments，projects也不要落下哦：[CS 229: Machine Learning](https://link.zhihu.com/?target=http%3A//cs229.stanford.edu/)

- 《[Pattern Classification](https://link.zhihu.com/?target=http%3A//cns-classes.bu.edu/cn550/Readings/duda-etal-00.pdf)》。 这本书几乎把所有模型放到一起挨个讲了一遍，虽然不如每个方法对应的专著那样深入，但是讲得还是挺透彻，啃完之后面对别人提到的任何机器学习算法都可以做到心中有数无压力。
- 《[Pattern Recognition and Machine Learning](https://link.zhihu.com/?target=http%3A//users.isr.ist.utl.pt/%7Ewurmd/Livros/school/Bishop%2520-%2520Pattern%2520Recognition%2520And%2520Machine%2520Learning%2520-%2520Springer%2520%25202006.pdf)》。这本书对贝叶斯方法的阐述个人觉得最佳。SVM那部分感觉讲的不太好，看不下去可以略过。看完之后搞generative model无压力。
- 《[统计机器学习](https://link.zhihu.com/?target=http%3A//pan.baidu.com/share/link%3Fuk%3D3208094820%26shareid%3D3392885063%26third%3D1%26adapt%3Dpc%26fr%3Dftw)》。不多说，中国人用中文写的好书。李航大神的良心大作。关键中文书的好处是轻便，不像英文书的大部头，走到哪里都能带着，简直居家旅行必备。
入门的书这几本感觉应该够了，看完了你就会自己发现想看的新的书了。很重要的一点是看的过程中，尤其是跟课的过程中，作业不要落下。要动手编程，自己实现几个模型。


**阶段三：登堂入室**     

看完了教材，现在又该回过头来把编程的能力提升一下了。阶段二里面公式推得飞起，可是你认真看过你写的code吗？结构怎么样？模块分得科不科学？如果让你重新写一遍，你会怎么去写？

结合着看几个大的framework，看看大神们是怎么实现自己之前实现过的那些算法的；他们的接口怎么设计，怎么设置模块。以及更为实用的，怎么使用这些framework。   

- scikit-learn。[scikit-learn: machine learning in Python](https://link.zhihu.com/?target=http%3A//scikit-learn.org/stable/)
- Deep Learning方面依旧是推荐一个基于Theano的framework：lasagne（[GitHub - Lasagne/Lasagne: Lightweight library to build and train neural networks in Theano](https://link.zhihu.com/?target=https%3A//github.com/Lasagne/Lasagne)）。

其余的库也值得了解。比如Torch，MXNET跟TensorFlow。个人觉得如果不是专精于开发，这几个当中精通一个，然后能看得懂别的几个，就可以了。因为个人能力所限，对其他三个库以及它们的“生态系统”了解不太多，缺乏横向比较也不敢推荐，还等大神出来补齐这方面的空白。对于Torch，TensorFlow跟Theano的性能倒是有一个直接的对比：[arxiv.org 的页面](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1605.02688v1.pdf)

看进去就会发现，一个framework的体量根本就远远超出一个人的精力所能涵盖的范围。对于大的库，甚至看完它的源代码都已是不可能。所以重要的是看人家的设计、用人家的接口，在实用中一点点了解深入。比如实现几个大的模型，改改其中的结构，再换个数据集跑一跑。

恭喜，在完成了这些之后你已经入门了。


**阶段四：心领神会**  

继续看书，看深入地讲某一方面的书。把之前过程中遇到的问题打破砂锅问到底。由于每本书都可以是迷死你几个月没商量的主，所以这一部分会花很长的时间，基本得和前一个或者后一个阶段同时进行。而且必须明白的一点是，这些书不可能全部看完，从跟自己最相关最想看的开始看吧，能啃一本是一本。推荐的书：

- 《Statistical Learning Theory》
- 《All of Statistics》
- 《Convex Optimization》
- 《Reinforcement Learning: An Introduction》
- 《Deep Learning》
- 《An Introduction to Information Retrieval》   

。。。。。。
推荐一个github repo，上面搜集了很多这方面的书：[GitHub: awesome machine learning](https://link.zhihu.com/?target=https%3A//github.com/josephmisiti/awesome-machine-learning/blob/master/books.md)

还有课程： 

- Fei-fei Li 关于ConvNets的课程，虽然她只上了一节课。。Stanford University CS231n: Convolutional Neural Networks for Visual Recognition
- 阿花狗的父亲之一，David Silver，给的关于reinforcement learning 的课程： http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching.html
- Hinton在coursera上著名的公开课：Neural Networks for Machine Learning
- PGM的祖师奶Daphne Koller给你讲PGM：Probabilistic Graphical Models
- NLP：Introduction to Natural Language Processing
- 自动机理论：Automata


**阶段五：融会贯通**  

到这里你会发现这时候你需要的东西那么多。而且不同的子方向深入之后所需要的东西都变得很不一样，不可能一一列出表来。然而有几件事情还是都一样的：

- **阅读文献**。主要就是arXiv。保证每周细读两篇paper，并且对arxiv上几个相关主题下每周放出来的新paper有个大概的了解（即看过标题或者abstract）。另外还可以在google scholar上关注几个领域内主要的大牛，然后设置新paper提醒。男神出新的paper了？让google贴心地推送到你的邮箱。

- **独立地思考问题**。尽信paper不如无paper。哪些paper你觉得有问题？哪些paper里面的主张跟你的想法不一样？如果要搞明白，怎样设计实验来分辨出谁对谁错？怎样在别人的paper里面找到破绽驳倒他们的主张？在这个过程中，自己的嘴会变得越发“刁钻刻薄”，当然也使得你写的paper更能挡住别人刁钻刻薄的追问。

- **跟数据说话**。就像F1的赛车一样，任你引擎跑到多少转任你空气动力学做得如何风骚，最后都得落实到四个轮子上才能把赛车的性能发挥出来。机器学习也一样，任你理论做的怎么漂亮，全得靠数据上实际跑出来的效果说话。所以要不断跟数据打交道，一个方法放上去不好使，那么为啥不好使？好使的时候中间结果应该是什么样的？哪些地方模型的表现暗示着模型没有抓住数据的一些特点？怎样的模型适合具有某一类特性的数据？要怎样改模型才可以让模型能够去学习到你想让它学的东西？用上你的改进后，模型的表现又变得怎样了？

- **夯实基础知识**。到这个时候基础知识是那么重要。那些原来看上去跟机器学习没关系的课都变得重要了起来。大学里的每门课（额尤其是数学课）都变得那么重要，你好想再去上一遍。。当然哪些基础知识需要夯实，不同的流派就有不同的风格：

1. 峨眉派                       
峨眉派的特点就是数学好。微分几何、凸优化、图论、流形、泛函分析、拓扑学。。数学你还会嫌学得太多吗？          
2. 武当派                       
武当派最不喜欢数学，喜欢靠intuition说话。祖师爷Hinton就是这么个人。心理学的背景，致力于搞明白“how the brain works”。Yoshua最近最看重的工作之一也是从神经科学得到启发，叫做biologically plausible deep learning。ConvNet也是受到对猫的神经元的研究的启发。事实上神经网络发展到现在，也没有个严格点数学理论去解释，倒是多看看神经科学的东西，也许会有新的发现。所以这个路子虽然野，却是大神辈出。补点神经科学方面的基础知识绝对不吃亏：Computational Neuroscience。就相当于是机器学习中的仿生学吧。                
3. 昆仑派                       
这一派也是数学好，只是他们概率统计特别强大，基本就靠这个吃饭。搞generative model，推bound。之所以把这一部分单列出来，是因为概率和统计在机器学习当中太重要了。           
4. 硬件派                 
“我们是搞硬件的可是那也阻当不了我们搞人工智能！我们搞加速算法、压缩模型、搞dedicated hardware。”       
5. 丐帮                            
“我没那么厉害的数学基础，但是我编程厉害啊！反正人工智能，我靠大量的实验压死别的派别！我用销魂的架构设计让大家都来用我写的代码！”                        


越写画风越不正经，就此打住。并不是说投靠了一个门派其他门派都不用管了，而是说自己门派的东西要比较全面地了解，了解到可以当作自己的背景；别的门派的也要了解些感兴趣的。每个人背景不一样，看问题的角度也不一样，不用也不可能做到样样精通。相比而言更有效的是跟不同背景的人多交流。好的idea经常就是在交流中爆出来的。

具体哪个派别厉害呢？江湖变幻莫测，又岂能一言蔽之。请时刻关注一年一度的华山论剑：NIPS、ICML、AAAI、ICLR。


**Rewferences**

[对人工智能有着一定憧憬的计算机专业学生可以阅读什么材料或书籍真正开始入门人工智能的思路和研究？ - 知乎](https://www.zhihu.com/question/44864396)