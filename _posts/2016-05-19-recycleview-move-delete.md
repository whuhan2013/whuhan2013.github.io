---
layout: post
title: RecyclerView拖拽排序和滑动删除实现
date: 2016-5-19
categories: blog
tags: [自定义控件]
description: RecyclerView拖拽排序和滑动删除实现
---   

### 效果图              


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoATZsTnqkSwiandQVR5WRar1ZS8KZwOMDzNkpRib095XPcLOlmicINBXmQ/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoxUbicAwSvTl6PMES1eZ7JnVO2D0654OYsAEcx7v7cnicPNONFLiaUXLTw/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)


### 如何实现   


那么是如何实现的呢？主要就要使用到ItemTouchHelper ，ItemTouchHelper 一个帮助开发人员处理拖拽和滑动删除的实现类，它能够让你非常容易实现侧滑删除、拖拽的功能。

实现的代码非常简单我们只需要两步：

实例化一个ItemTouchHelper            
关联到RecyclerView         


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoG2YHWgwAcWiacHanqNJqQtvjP2HC2S6ia1icQ1xvfLibIPibVyqoszlTyvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



恩，就是这么简单。

构造方法中需要一个ItemTouchHelper.Callback，ItemTouchHelper会在拖拽的时候回调Callback中相应的方法，我们只需在Callback中实现自己的逻辑就可以了。

自定义一个类继承实现ItemTouchHelper.Callback接口，需要实现以下方法：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eobIQlfVODFnKu7eVgcNlXxTGAxZGObm3YHDwribic7bQVpkb6jGFYNg3w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


getMovementFlags用于设置是否处理拖拽事件和滑动事件，以及拖拽和滑动操作的方向，有以下两种情况：

如果是列表类型的RecyclerView，拖拽只有UP、DOWN两个方向       
如果是网格类型的则有UP、DOWN、LEFT、RIGHT四个方向           

该方法需要编写的代码如下：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eo6ZAxGh6jDD9icE4HbyZ2PerBC6RYXE3tFcsld7bFhSL4BIwktViamILw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


dragFlags 是拖拽标志，swipeFlags是滑动标志，我们把swipeFlags 都设置为0，暂时不考虑滑动相关操作。

如果我们设置了相关的dragFlags ，那么当我们长按item的时候就会进入拖拽并在拖拽过程中不断回调onMove()方法，我们就在这个方法里获取当前拖拽的item和已经被拖拽到所处位置的item的ViewHolder，有了这2个ViewHolder，我们就可以交换他们的数据集并调用Adapter的notifyItemMoved方法来刷新item。


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoOxqtzIl5BRKdKIogGML3MU8fMfSWmYMh0ibJs8icsh0GEydawcqbFrfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


这里的mDatas其实就是Adapter对应的数据集，我们在改变Item位置的同时当然不能忘了数据集需要同步的改变。

到这里，已经可以拖拽了，基本的效果如下：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eodiaYpdojW6QE5ibodKpEfVTnBHwbmewCw98JFKzM3PDn5N22wB6XFiakA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

是不是很神奇，这么复杂的特效，竟然写了几行代码就搞定了~

但是拖拽的时候我们拖拽的对象不能高亮显示，这是不友好的，我们希望拖拽的Item在拖拽的过程中背景颜色加深，这样就需要继续重写下面两个方法：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoqVy7cUwDGp3mGdsPrrdFAzqTt8YtIHNQm3zvJAI8PYP158UGj5m9ibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


我们在开始拖拽的时候给item添加一个背景色，然后在拖拽完成的时候还原：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eolX7EVHNN7EBGSUsjzv69n6pP0gqhGhHaSqrHRMtf5DiccUVkYMWVdHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


OK，这样就完成了Item的拖拽排序，简单看下现在的效果：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eobr1YNiaFMpbvq4IibtiaoO3ibiaY9mXfcHZNa3DWrK0vCnnt5F8r11gn1oA/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)


### 更加复杂的需求


上面的代码完成了基本功能，但实际的产品需要往往可能会有些不一样，比如说，产品希望，有一些item可以拖拽，一些item无法拖拽，就如上图的“更多”是无法拖拽的。这个咋办呢？

其实在上面我们实现的Callback类中有一个方法我们没有重写：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoAfLUNn0TDFickWqCicQLqQAHyDtdH04GRmLD78Aw1ibUCWO24P8UkhcHw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

这个方法是为了告诉ItemTouchHelper是否需要RecyclerView支持长按拖拽，默认返回是ture（即支持），理所当然我们要支持，所以我们没有重写，因为默认true。但是这样做是默认全部的item都可以拖拽，怎么实现部分item拖拽呢，查阅isLongPressDragEnabled方法的源码发现，上面的注释上写着：

```
Default value returns true but you may want to disable this if you want to 
start dragging on a custom view touch using 
{@link #startDrag(ViewHolder)}.
```

意思是如果你想自定义触摸view，那么就使用startDrag(ViewHolder)方法。

原来如此，我们可以在item的长按事件中得到当前item的ViewHolder ，然后调用ItemTouchHelper.startDrag(ViewHolder vh)就可以实现拖拽了，那就这么办：

首先我们重写isLongPressDragEnabled返回false，我们要自己调用拖拽过程：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoZqc349XEXGKndeFy8omlRD3vXUg7OmmkhKCbS8WSCDmxQyuFtnWYgg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

接着我们给RecyclerView添加item长按事件，判断item是否是最后一个（最后一个是“更多”），不是则开始拖拽。

但是，我们都知道RecyclerView并没有提供OnItemLongClickListener，这个问题我们已经在上一篇推送的文章中给出了解决方案，就是使用OnItemTouchListener，然后识别触摸手势，这里给上传送门：[RecyclerView添加ItemClickListener的方案](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820134&idx=1&sn=58103e352e5269159778d35dc36ed207&scene=21#wechat_redirect)，我就直接使用上一篇的成果，不重复讲了（这里你可以使用直接的OnItemClickListener的方案）：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoSibUvbz0bDqQCJiczHg4psjABbX7Fn4NGMWOerDYSiaQyeuYShFszXmPw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


### 额外的功能


### 保存位置

关闭页面以后再打开，又恢复到了初始化的位置，所以就需要保存调整的位置到本地，下次初始化的时候读取位置。 

保存位置应该由开发者自己实现，因为每个人本地化数据的方式都不一样，我这里做一个简单的实现，使用了开源的ACache类，两个方法，搞定：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eobxN6eXyj5UibyS8M3P1sMTuCIF7SE8SC1SXczoXT4kLwoGaUvq2EkKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)


在clearView方法（拖拽完成）中调用存储方法，在页面初始化数据是调用读取方法，具体代码文后会给出下载链接。

### 开始拖拽时震动

支付宝的拖拽网格在长按后开始拖拽时会有一次短时间的震动提示用户开始拖拽了，很友好的交互，我们也加一个：

添加权限：

```
<uses-permission android:name="android.permission.VIBRATE" />
```


在开始拖拽时添加下面代码：

![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoPIJh1zlvB0v7fK1Pib19kJLT7F23Wz4B4Jl6pzGoMicSYsm3Jqz9g79w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

### 滑动删除

  
我看到接口里面还有个onSwipe方法，没有使用，没错，这个可用于滑动删除。

仅仅需要几行代码实现滑动删除：


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eoURvBd7ic6GfZSO4AOTf57sLA4ib2S00LnicBPP67yxEz5zvLf2919WQlQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

恩，就这样就行了？

还不行，还有标志位没设置


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eogzhchLXsLVmIzcf7BqYJfeoibxWDia00huiaSjmWbFn0t2K0gngtARevw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)



我们针对非表格式的布局设置了swipeFlags为STRAT和END，那么LinearLayoutManager肯定是支持的，看下效果


![](http://mmbiz.qpic.cn/mmbiz/MOu2ZNAwZwPegXTpG6dwKNNj2YZJ40eobNeAaxNuU2JwUlv9ZtvTcswVVjYRPmsfej9CibYzlyibLYW3bmE6F2dQ/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

### fragment中的onCreateView和onViewCreated的区别和联系


onViewCreated在onCreateView执行完后立即执行。
onCreateView返回的就是fragment要显示的view。

### 参考链接

[fragment中的onCreateView和onViewCreated的区别和联系_百度知道](http://zhidao.baidu.com/link?url=VT0vbmRTIqvTsyGgMvKS8XjB57tzuP28im4hSt28sWFMu6iYpTtU6eV_cr8tFWFASKXtkdj9796EMj0xpd4GP8JFKcRLySbI_MylyxuvLK3)

### 源代码 

[http://download.csdn.net/detail/liaoinstan/9494601](http://download.csdn.net/detail/liaoinstan/9494601)


### 原文链接

[使用ItemTouchHelper轻松实现RecyclerView拖拽排序和滑动删除](http://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650820215&idx=1&sn=7a7da6210f7f0b975674422fa4b159ef&scene=0#wechat_redirect)

