---
layout: post
title: matplotlib绘图总结
date: 2016-9-16
categories: blog
tags: [数据分析]
description: matplotlib绘图总结
---

本文作为学习过程中对matplotlib一些常用知识点的整理，方便查找。

#### 类MATLAB API

最简单的入门是从类 MATLAB API 开始，它被设计成兼容 MATLAB 绘图函数。

```
from pylab import *

from numpy import *
x = linspace(0, 5, 10)
y = x ** 2

figure()
plot(x, y, 'r')
xlabel('x')
ylabel('y')
title('title')
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_9.png)

创建子图，选择绘图用的颜色与描点符号:

```
subplot(1,2,1)
plot(x, y, 'r--')
subplot(1,2,2)
plot(y, x, 'g*-');
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_10.png)


linspace表示在0到5之间用10个点表示，plot的第三个参数表示画线的颜色与样式

此类 API 的好处是可以节省你的代码量，但是我们并不鼓励使用它处理复杂的图表。处理复杂图表时， matplotlib 面向对象 API 是一个更好的选择。

#### matplotlib 面向对象 API  

使用面向对象API的方法和之前例子里的看起来很类似，不同的是，我们并不创建一个全局实例，而是将新建实例的引用保存在 fig 变量中,如果我们想在图中新建一个坐标轴实例，只需要 调用 fig 实例的 add_axes 方法：

```
import matplotlib.pyplot as plt
from pylab import *
x = linspace(0, 5, 10)
y = x ** 2

fig = plt.figure()

axes = fig.add_axes([0.1, 0.1, 0.8, 0.8]) # left, bottom, width, height (range 0 to 1)

axes.plot(x, y, 'r')

axes.set_xlabel('x')
axes.set_ylabel('y')
axes.set_title('title')

plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_11.png)      

尽管会写更多的代码，好处在于我们对于图表的绘制有了完全的控制权，可以很容易地多加一个坐标轴到图中：


```
import matplotlib.pyplot as plt
from pylab import *
x = linspace(0, 5, 10)
y = x ** 2

fig = plt.figure()

axes = fig.add_axes([0.1, 0.1, 0.8, 0.8]) # left, bottom, width, height (range 0 to 1)
axes2 = fig.add_axes([0.2, 0.5, 0.4, 0.3]) # inset axes

axes.plot(x, y, 'r')

axes.set_xlabel('x')
axes.set_ylabel('y')
axes.set_title('title')

# insert
axes2.plot(y, x, 'g')
axes2.set_xlabel('y')
axes2.set_ylabel('x')
axes2.set_title('insert title');

plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_12.png)  


如果我们不在意坐标轴在图中的排放位置️，那么就可以使用matplotlib的布局管理器了，我最喜欢的是subplots，使用方式如下：

```
import matplotlib.pyplot as plt
from pylab import *
x = linspace(0, 5, 10)
y = x ** 2

fig, axes = plt.subplots(nrows=1, ncols=2)

for ax in axes:
    ax.plot(x, y, 'r')
    ax.set_xlabel('x')
    ax.set_ylabel('y')
    ax.set_title('title')

fig.tight_layout()


plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_13.png)


**图表尺寸，长宽比 与 DPI**

在创建 Figure 对象的时候，使用figsize 与 dpi 参数能够设置图表尺寸与DPI， 创建一个800*400像素，每英寸100像素的图就可以这么做：

```
fig = plt.figure(figsize=(8,4), dpi=100)


<matplotlib.figure.Figure at 0x4cbd390>
```

同样的参数也可以用在布局管理器上:

```
fig, axes = plt.subplots(figsize=(12,3))

axes.plot(x, y, 'r')
axes.set_xlabel('x')
axes.set_ylabel('y')
axes.set_title('title');
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_14.png)


**保存图表**

可以使用 savefig 保存图表

```
fig.savefig("filename.png")
这里我们也可以有选择地指定DPI，并且选择不同的输出格式：

fig.savefig("filename.png", dpi=200)
```

有哪些格式？哪种格式能获得最佳质量？

Matplotlib 可以生成多种格式的高质量图像，包括PNG，JPG，EPS，SVG，PGF 和 PDF。如果是科学论文的话，我建议尽量使用pdf格式。 (pdflatex 编译的 LaTeX 文档使用 includegraphics 命令就能包含 PDF 文件)。 一些情况下，PGF也是一个很好的选择。


#### 图例，轴标 与 标题    

现在我们已经介绍了如何创建图表画布以及如何添加新的坐标轴实例，让我们看一看如何加上标题，轴标和图例

标题

每一个坐标轴实例都可以加上一个标题，只需调用坐标轴实例的 set_title 方法:

```
ax.set_title("title");
```

轴标

类似的， set_xlabel 与 set_ylabel 可以设置坐标轴的x轴与y轴的标签。

```
ax.set_xlabel("x")
ax.set_ylabel("y");
```

图例

有两种方法在图中加入图例。一种是调用坐标轴对象的 legend 方法，传入与之前定义的几条曲线相对应地图例文字的 列表/元组：

ax.legend(["curve1", "curve2", "curve3"]);

不过这种方式容易出错，比如增加了新的曲线或者移除了某条曲线。更好的方式是在调用 plot方法时使用 label="label text" 参数，再调用 legend 方法加入图例：

```
ax.plot(x, x**2, label="curve1")
ax.plot(x, x**3, label="curve2")
ax.legend();
```


legend 还有一个可选参数 loc 决定画出图例的位置，详情见：http://matplotlib.org/users/legend_guide.html#legend-location

最常用的值如下：

```
ax.legend(loc=0) # let matplotlib decide the optimal location
ax.legend(loc=1) # upper right corner
ax.legend(loc=2) # upper left corner
ax.legend(loc=3) # lower left corner
ax.legend(loc=4) # lower right corner
# .. many more options are available

=> <matplotlib.legend.Legend at 0x4c863d0>
```


下面这个例子同时包含了标题，轴标，与图例的用法：

```
import matplotlib.pyplot as plt
from pylab import *
x = linspace(0, 5, 10)
y = x ** 2

fig, ax = plt.subplots()

ax.plot(x, x**2, label="y = x**2")
ax.plot(x, x**3, label="y = x**3")
ax.legend(loc=2); # upper left corner
ax.set_xlabel('x')
ax.set_ylabel('y')
ax.set_title('title');

plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_15.png)


**格式化文本，LaTeX，字体大小，字体类型**

Matplotlib 对 LaTeX 提供了很好的支持。我们只需要将 LaTeX 表达式封装在 $ 符号内，就可以在图的任何文本中显示了，比如 "$y=x^3$" 。

不过这里我们会遇到一些小问题，在 LaTeX 中我们常常会用到反斜杠，比如 \alpha 来产生符号 $\alpha$ 。但反斜杠在 python 字符串中是有特殊含义的。为了不出错，我们需要使用原始文本，只需要在字符串的前面加个r就行了，像是 r"\alpha" 或者 r'\alpha'：

```
import matplotlib.pyplot as plt
from pylab import *
x = linspace(0, 5, 10)
y = x ** 2

fig, ax = plt.subplots()

ax.plot(x, x**2, label=r"$y = \alpha^2$")
ax.plot(x, x**3, label=r"$y = \alpha^3$")
ax.legend(loc=2) # upper left corner
ax.set_xlabel(r'$\alpha$', fontsize=18)
ax.set_ylabel(r'$y$', fontsize=18)
ax.set_title('title');


plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_16.png)


我们可以更改全局字体大小或者类型:

```
from matplotlib import rcParams
rcParams.update({'font.size': 18, 'font.family': 'serif'})
```

STIX 字体是一种好选择：

```
matplotlib.rcParams.update({'font.size': 18, 'font.family': 'STIXGeneral', 'mathtext.fontset': 'stix'})
```


我们也可以将图中的文本全用 Latex 渲染：

```
matplotlib.rcParams.update({'font.size': 18, 'text.usetex': True})
```


**设置颜色，线宽 与 线型**

颜色

有了matplotlib，我们就有很多方法能够定义线的颜色和很多其他图形元素。首先，我们可以使用类MATLAB语法，'b' 代表蓝色，'g' 代表绿色，依此类推。matplotlib同时也支持 MATLAB API 选择线型所使用的方式：比如 'b.-' 意味着蓝线标着点：

```
# MATLAB style line color and style 
ax.plot(x, x**2, 'b.-') # blue line with dots
ax.plot(x, x**3, 'g--') # green dashed line

fig

=> [<matplotlib.lines.Line2D at 0x4985810>]
```


我们也可以以颜色的名字或者RGB值选择颜色，alpha参数决定了颜色的透明度：

```
fig, ax = plt.subplots()

ax.plot(x, x+1, color="red", alpha=0.5) # half-transparant red
ax.plot(x, x+2, color="#1155dd")        # RGB hex code for a bluish color
ax.plot(x, x+3, color="#15cc55")        # RGB hex code for a greenish color

fig

=> [<matplotlib.lines.Line2D at 0x4edbd10>]
```

![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328255867.png/wm)


**线与描点风格**

linewidth 或是 lw 参数改变线宽。 linestyle 或是 ls 参数改变线的风格。

```
fig, ax = plt.subplots(figsize=(12,6))

ax.plot(x, x+1, color="blue", linewidth=0.25)
ax.plot(x, x+2, color="blue", linewidth=0.50)
ax.plot(x, x+3, color="blue", linewidth=1.00)
ax.plot(x, x+4, color="blue", linewidth=2.00)

# possible linestype options ‘-‘, ‘–’, ‘-.’, ‘:’, ‘steps’
ax.plot(x, x+5, color="red", lw=2, linestyle='-')
ax.plot(x, x+6, color="red", lw=2, ls='-.')
ax.plot(x, x+7, color="red", lw=2, ls=':')

# custom dash
line, = ax.plot(x, x+8, color="black", lw=1.50)
line.set_dashes([5, 10, 15, 10]) # format: line length, space length, ...

# possible marker symbols: marker = '+', 'o', '*', 's', ',', '.', '1', '2', '3', '4', ...
ax.plot(x, x+ 9, color="green", lw=2, ls='*', marker='+')
ax.plot(x, x+10, color="green", lw=2, ls='*', marker='o')
ax.plot(x, x+11, color="green", lw=2, ls='*', marker='s')
ax.plot(x, x+12, color="green", lw=2, ls='*', marker='1')

# marker size and color
ax.plot(x, x+13, color="purple", lw=1, ls='-', marker='o', markersize=2)
ax.plot(x, x+14, color="purple", lw=1, ls='-', marker='o', markersize=4)
ax.plot(x, x+15, color="purple", lw=1, ls='-', marker='o', markersize=8, markerfacecolor="red")
ax.plot(x, x+16, color="purple", lw=1, ls='-', marker='s', markersize=8, 
        markerfacecolor="yellow", markeredgewidth=2, markeredgecolor="blue")

fig
```


![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328263233.png/wm)


**控制坐标轴的样式**

坐标轴样式也是通常需要自定义的地方，像是标号或是标签的位置或是字体的大小等。

图的范围

我们想做的第一件事也许是设置坐标轴的范围，可以使用 set_ylim 或是 set_xlim 方法或者 axis('tight') 自动将坐标轴调整的紧凑 The first thing we might want to configure is the ranges of the axes. We can do this using the set_ylim and set_xlim methods in the axis object, or axis('tight') for automatrically getting "tightly fitted" axes ranges:

```
fig, axes = plt.subplots(1, 3, figsize=(12, 4))

axes[0].plot(x, x**2, x, x**3)
axes[0].set_title("default axes ranges")

axes[1].plot(x, x**2, x, x**3)
axes[1].axis('tight')
axes[1].set_title("tight axes")

axes[2].plot(x, x**2, x, x**3)
axes[2].set_ylim([0, 60])
axes[2].set_xlim([2, 5])
axes[2].set_title("custom axes range");

fig
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_17.png)


**对数刻度**

也可以将轴的刻度设置成对数刻度，调用 set_xscale 与 set_yscale 设置刻度，参数选择 "log" ：

```
fig, axes = plt.subplots(1, 2, figsize=(10,4))

axes[0].plot(x, x**2, x, exp(x))
axes[0].set_title("Normal scale")

axes[1].plot(x, x**2, x, exp(x))
axes[1].set_yscale("log")
axes[1].set_title("Logarithmic scale (y)");

fig
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_18.png)



**自定义标号位置与符号**

set_xticks 与 set_yticks 方法可以显示地设置标号的位置， set_xticklabels 与 set_yticklabels 为每一个标号设置符号：

```
fig, ax = plt.subplots(figsize=(10, 4))

ax.plot(x, x**2, x, x**3, lw=2)

ax.set_xticks([1, 2, 3, 4, 5])
ax.set_xticklabels([r'$\alpha$', r'$\beta$', r'$\gamma$', r'$\delta$', r'$\epsilon$'], fontsize=18)

yticks = [0, 50, 100, 150]
ax.set_yticks(yticks)
ax.set_yticklabels(["$%.1f$" % y for y in yticks], fontsize=18); # use LaTeX formatted labels

fig

=> [<matplotlib.text.Text at 0x5d75c90>,
    <matplotlib.text.Text at 0x585fe50>,
    <matplotlib.text.Text at 0x575c090>,
    <matplotlib.text.Text at 0x599e610>]
```

![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328338951.png/wm)


**科学计数法**

如果轴上涉及非常大的数，最好使用科学计数法：

```
fig, ax = plt.subplots(1, 1)

ax.plot(x, x**2, x, exp(x))
ax.set_title("scientific notation")

ax.set_yticks([0, 50, 100, 150])

from matplotlib import ticker
formatter = ticker.ScalarFormatter(useMathText=True)
formatter.set_scientific(True) 
formatter.set_powerlimits((-1,1)) 
ax.yaxis.set_major_formatter(formatter) 

fig
```


![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328359594.png/wm)


**轴上数与标签的间距**

```
# distance between x and y axis and the numbers on the axes
rcParams['xtick.major.pad'] = 5
rcParams['ytick.major.pad'] = 5

fig, ax = plt.subplots(1, 1)

ax.plot(x, x**2, x, exp(x))
ax.set_yticks([0, 50, 100, 150])

ax.set_title("label and axis spacing")

# padding between axis label and axis numbers
ax.xaxis.labelpad = 5
ax.yaxis.labelpad = 5

ax.set_xlabel("x")
ax.set_ylabel("y");

fig
```

**调整坐标轴的位置：**

```

fig, ax = plt.subplots(1, 1)

ax.plot(x, x**2, x, exp(x))
ax.set_yticks([0, 50, 100, 150])

ax.set_title("title")
ax.set_xlabel("x")
ax.set_ylabel("y")

fig.subplots_adjust(left=0.15, right=.9, bottom=0.1, top=0.9);

fig

```


**坐标轴网格**

grid 方法可以打开关闭网格线，也可以自定义网格的样式：

```
fig, axes = plt.subplots(1, 2, figsize=(10,3))

# default grid appearance
axes[0].plot(x, x**2, x, x**3, lw=2)
axes[0].grid(True)

# custom grid appearance
axes[1].plot(x, x**2, x, x**3, lw=2)
axes[1].grid(color='b', alpha=0.5, linestyle='dashed', linewidth=0.5)

fig
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_19.png)


**轴**

我们也可以改变轴的属性:

```
fig, ax = plt.subplots(figsize=(6,2))

ax.spines['bottom'].set_color('blue')
ax.spines['top'].set_color('blue')

ax.spines['left'].set_color('red')
ax.spines['left'].set_linewidth(2)

# turn off axis spine to the right
ax.spines['right'].set_color("none")
ax.yaxis.tick_left() # only ticks on the left side

fig
```


**双坐标轴**

twinx 与 twiny 函数能设置双坐标轴：


```
fig, ax1 = plt.subplots()

ax1.plot(x, x**2, lw=2, color="blue")
ax1.set_ylabel(r"area $(m^2)$", fontsize=18, color="blue")
for label in ax1.get_yticklabels():
    label.set_color("blue")

ax2 = ax1.twinx()
ax2.plot(x, x**3, lw=2, color="red")
ax2.set_ylabel(r"volume $(m^3)$", fontsize=18, color="red")
for label in ax2.get_yticklabels():
    label.set_color("red")

fig
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_20.png)


**设置坐标原点在（0，0）点**

```
fig, ax = plt.subplots()

ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')

ax.xaxis.set_ticks_position('bottom')
ax.spines['bottom'].set_position(('data',0)) # set position of x spine to x=0

ax.yaxis.set_ticks_position('left')
ax.spines['left'].set_position(('data',0))   # set position of y spine to y=0

xx = np.linspace(-0.75, 1., 100)
ax.plot(xx, xx**3);

fig
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_21.png)



#### 其他 2D 图表风格

包括一般的 plot 方法, 还有很多其他函数能够生成不同类型的图表，详情请见 http://matplotlib.org/gallery.html 这里列出其中几种比较常见的函数方法。


```
n = array([0,1,2,3,4,5])


fig, axes = plt.subplots(1, 4, figsize=(12,3))

axes[0].scatter(xx, xx + 0.25*randn(len(xx)))
axes[0].set_title("scatter")

axes[1].step(n, n**2, lw=2)
axes[1].set_title("step")

axes[2].bar(n, n**2, align="center", width=0.5, alpha=0.5)
axes[2].set_title("bar")

axes[3].fill_between(x, x**2, x**3, color="green", alpha=0.5);
axes[3].set_title("fill_between");

fig
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_22.png)


```
# polar plot using add_axes and polar projection
fig = plt.figure()
ax = fig.add_axes([0.0, 0.0, .6, .6], polar=True)
t = linspace(0, 2 * pi, 100)
ax.plot(t, t, color='blue', lw=3);
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_23.png)



```
# A histogram
n = np.random.randn(100000)
fig, axes = plt.subplots(1, 2, figsize=(12,4))

axes[0].hist(n)
axes[0].set_title("Default histogram")
axes[0].set_xlim((min(n), max(n)))

axes[1].hist(n, cumulative=True, bins=50)
axes[1].set_title("Cumulative detailed histogram")
axes[1].set_xlim((min(n), max(n)));

fig
```

hist的参数含义    

x : (n,) array or sequence of (n,) arrays

这个参数是指定每个bin(箱子)分布的数据,对应x轴

bins : integer or array_like, optional

这个参数指定bin(箱子)的个数,也就是总共有几条条状图

normed : boolean, optional

If True, the first element of the return tuple will be the counts normalized to form a probability density, i.e.,n/(len(x)`dbin)

这个参数指定密度,也就是每个条状图的占比例比,默认为1

color : color or array_like of colors or None, optional

这个指定条状图的颜色

参见：[ hist的使用 ](http://blog.csdn.net/u013571243/article/details/48998619)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_24.png)



**饼状图**


```
import matplotlib.pyplot as plt

labels='frogs','hogs','dogs','logs'
sizes=15,20,45,10
colors='yellowgreen','gold','lightskyblue','lightcoral'
explode=0,0.1,0,0
plt.pie(sizes,explode=explode,labels=labels,colors=colors,autopct='%1.1f%%',shadow=True,startangle=50)
plt.axis('equal')
plt.show()
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/figure_8.png)    



**文本注释**

text 函数可以做文本注释，且支持 LaTeX 格式：

```
fig, ax = plt.subplots()

ax.plot(xx, xx**2, xx, xx**3)

ax.text(0.15, 0.2, r"$y=x^2$", fontsize=20, color="blue")
ax.text(0.65, 0.1, r"$y=x^3$", fontsize=20, color="green");

fig
```


**带有多子图与插图的图**

fig.add_axes 在图中加入新坐标轴

subplots， subplot2grid，gridspec等 子图布局管理器

**subplots**


```
fig, ax = plt.subplots(2, 3)
fig.tight_layout()

fig
```

![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328674433.png/wm)


**subplot2grid**

```
fig = plt.figure()
ax1 = plt.subplot2grid((3,3), (0,0), colspan=3)
ax2 = plt.subplot2grid((3,3), (1,0), colspan=2)
ax3 = plt.subplot2grid((3,3), (1,2), rowspan=2)
ax4 = plt.subplot2grid((3,3), (2,0))
ax5 = plt.subplot2grid((3,3), (2,1))
fig.tight_layout()

fig
```

![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328683200.png/wm)




**颜色映射图与轮廓图**

颜色映射图与轮廓图适合绘制两个变量的函数。


![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328762835.png/wm)

![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1080timestamp1468328771398.png/wm)



matplotlib也可以画3D图，这里暂且略过。

**参考：**[matplotlib - 2D 与 3D 图的绘制（上）](https://www.shiyanlou.com/courses/348/labs/1080/document)



































