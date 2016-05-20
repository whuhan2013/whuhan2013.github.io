---
layout: post
title: BaseActivity与BaseFragment的封装
date: 2016-5-20
categories: blog
tags: [android]
description: BaseActivity与BaseFragment的封装
---   

这篇博客主要是从BaseActivity与BaseFragment的封装开始，总结我们在实战开发中关于Fragment的注意事项以及心得体会。

先看以下效果图：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBAvVkHXQO9jSDEM8PE9SQMTI8momr06zRw2oq7KZKW50TIIz52APnOg/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)


这里模拟的是用户登录模块，你可能会说，很普通的效果嘛，这有啥。嘿嘿，那我要告诉你的是，这么多模块仅仅由两个Activity构成的。等你从头到尾看完这篇博客，你就会惊叹其中的奥秘了。废话不多说，开始。

本案例属于多模块Activity+多Fragment，下面简单介绍下概念。

多模块Activity+多Fragment 是开发APP非常适合的架构，相对于多Activity，这种架构APP占用内存降低，性能提升；相对于单Activity+多Fragment，这种开发起来逻辑相对简单，不容易出错。

对于多模块Activity+多Fragment，这里有两个概念需要我们了解一下：

- 同级式Fragment： 比如QQ的主界面，消息，联系人，动态，这三个Fragment就属于同级关系，我们平时项目中主界面的Fragment也是属于同级Fragment 
- 流程式Fragment：比如我这个示例Demo，可以理解为用户账户流程，可以包括：登录/注册模块----忘记/找回密码模块----用户协议模块，这些Fragent就是属于流程式Fragment

我的示例Demo使用的是流程式Fragment，结合今天的主题----BaseActivity与BaseFragment的封装，我们一探究竟。


### BaseActivity的封装


首先看一下代码：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBZ735uuNGEy9he5tuhCFjmpjNn927BfChaMG5Hpfsgwqb83oZ6PBMUg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



1. 两个必须实现的抽象方法，获取布局文件Layout的resource ID，获取布局文件中Fragment的ID          
     
2. 添加fragment：开启一个事物,替换了当前layout容器中的由getFragmentContentId()标识的fragment。通过调用 addToBackStack(String tag), replace事务被保存到back stack, 因此用户可以回退事务,并通过按下BACK按键带回前一个fragment，如果没有调用 addToBackStack(String tag), 那么当事务提交后, 那个fragment会被销毁,并且用户不能导航回到它。其中参数tag将作为本次加入BackStack的Transaction的标志。commitAllowingStateLoss()，这种提交是允许发生异常时状态值丢失的情况下也能正常提交事物   
         
3. 移除fragment：与addToBackStack()相对应的接口方法是popBackStack()，调用该方法后会将事务操作插入到FragmentManager的操作队列，轮询到该事务时开始执行。这里进行了一下判断，获取回退栈中所有事务数量，大于1的时候，执行回退操作，等于1的时候，代表当前Activity只剩下一个Fragment，直接finish()当前Activity即可                          
    
4. 监听返回键的返回事件，当事务数量等于1的时候，直接finish()


### 进一步封装AppActivity

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaB93VGpoMfUn8Fz084jea5ia1uXVh4uoiclDxqNWL2FDbichgvKiawY2Fzbw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


1. 一个必须实现的抽象方法来获取当前Activity应该显示的第一个Fragment         
2. 获取intent的方法，在需要传递/接受数据的Activity实现      
3. 在Activity的onCreate()方法中拿到intent，添加fragment

不过这样的封装是需要制定一个布局文件的，activity_base.xml布局文件代码为：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBdUJqqOhI37HpcXBlc7C4ZDNsiarTwUP34dSe04CKG1xm1DTs9e8Cx5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


### BaseFragment的封装


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBZSEU8hKliaztcrqXWfaUuyohVwBlh8UYqanVIRtuQZ2Bmlm1IKLjUFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


在APP运行在后台的时候，系统资源紧张的时候会导致后台的Activity被销毁，可能会带来一些问题，其中之一就是Fragment调用getActivity()的地方却返回null，报了空指针异常。

解决办法就是在Fragment基类里设置一个Activity mActivity的全局变量，在onAttach(Activity activity)里赋值，使用mActivity代替getActivity()。


### 封装后的使用


BaseActivity与BaseFragment的封装都已经完成，接下来就是具体在项目中的使用了，这里分两种情况。

- 第一种情况：没有参数的传递

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaB6hcicWlW9C73LmLcsWppaGDfPeb1LnvlhQph4Hia2VfvGmpYTSOp7hoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

示例Demo中的主界面MainActivity，可以看到代码相当的精简，对应的MainFragment代码如下：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBMrcEAoUVibKBImr7XfSB9PUPmJibfwGOJDnchIgErUljF58Qk8jJt1Eg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



很简单的业务逻辑，点击第一个按钮跳转到LoginActivity，点击第二个按钮跳转到注册模块，布局文件代码就不贴了。

- 第二种情况：有参数的传递

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBSukHfhibW60DznDYZrYJfGrOSJ9ESY5KeQV7nTvMMHyYot9SWt4T7cQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



可以看到，LoginActivity与之前不一样的是，重写了handleIntent()这个方法来获取传递过来的数据，重要的一点，创建Fragment的时候传递了一个参数。 代码很简单，即通过Arguments传递参数。


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaB3rqRo5xhNyStVsHXA2R7AUIJvc6pqBEp2nrY54boxzFQQyoraOsiahQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



给Fragment添加newInstance方法，将需要的参数传入，设置到bundle中，然后setArguments(bundle)，最后在onCreate中进行获取。

这种使用arguments来创建Fragment的方法，强烈推荐使用：

这样就完成了Fragment和Activity间的解耦,使用Fragment的一个很大的原因，就是为了复用。这一点在我主界面点击第二个按钮跳转到注册界面有所体现
对Fragment传递数据，建议使用setArguments(Bundle args)，而后在onCreate中使用getArguments()取出，在 内存不足导致异常时，系统会帮你保存数据，不会造成数据的丢失。和Activity的Intent原理一致。
使用newInstance(参数) 创建Fragment对象，优点是调用者只需要关系传递的哪些数据，而无需关心传递数据的Key是什么。

其他界面大同小异，大家可以在此自由发挥。关于流程式Fragment，就先到这里，看看同级式Fragment应该注意的问题。


### hide()与show()导致的Fragment重叠

同级式Fragment在内存不足导致的异常情况下，会出现重叠现象，处理方法是在基类的Activity中onSaveInstanceState()内保存当前所在Fragment的tag或者下标，在onCreate()是恢复的时候，隐藏其它Fragment。

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBSIiaMWZXZnJibtVgdnmFr71USiaS5iaYcecBUsC8n8FbrjU7JvfI9H7upw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)



假设我们存在三个同级Fragment，目前Fragment为ContactFragment，此时切换到后台，由于内存因素导致Activity重建，我们就可以通过上述代码解决Fragment重叠以及回到ContactFragment页面。

当然要记得在onSaveInstanceState存储下当前页面信息


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPfwI0yjhrIBeUhAVOpFniaBjv28zmSshHhsScibkEH4zmXxMibrj8edR8ibibhnvNxGZ7GYwFFyribibhCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


### 推荐阅读

- [Fragment全解析系列（一）：那些年踩过的坑](http://www.jianshu.com/p/d9143a92ad94)
- [Fragment全解析系列（二）：正确的使用姿势](http://www.jianshu.com/p/fd71d65f0ec6)
- [Fragment之我的解决方案：Fragmentation](http://www.jianshu.com/p/38f7994faa6b)
- [Android Fragment 你应该知道的一切](http://blog.csdn.net/lmj623565791/article/details/42628537)


### 参考链接

[从BaseActivity与BaseFragment的封装谈起](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820216&idx=1&sn=64d0380fedf947c7adefc6d3eb722301&scene=0#wechat_redirect)

