---
layout: post
title: 村庄尾字图实现
date: 2016-10-24
categories: blog
tags: [地图开发]
description: 村庄尾字图实现
---

在找相同村名称的时候，我们可以发现不同的村名称分布非常有特点，比如以湾结尾的村名分布区域大致相同，由此想到如果按照村庄名称的尾字出图将会是什么效果呢？这才有了中国村庄名称尾字图，按照以村、庄、湾、屯、岗等为结尾来画尾字图，基于qgis实现。

**数据来源** 

[村庄数据下载地址](https://geohey.com/gallery/data/b3230ab570924d729c598fc0ea5281a5)       

[中国的省级行政区划下载地址](http://geohey.com/gallery/data/administration_division_province_cn)


**数据格式**

下载的数据分为shp和kmz两种格式文件，其中shp格式为GIS领域通用的一种空间数据格式，而kmz为Google Earth的文件格式，这两种格式QGIS都支持

#### QGIS设置

打开QGIS看到下面的界面，左侧的layer panel 是图层列表，中间那个大的panel是地图窗口，右侧的是文件流浪窗口，在开始加载数据之前，需要进行一些设置：

背景颜色设置：现在看到地图窗口是白色背景，需要将其设置成黑色，打开Settings——>Options菜单，在Canvas & Legend选项卡中将Background color设置成黑色，如下图所示，点击OK后可能颜色还没有变，你需要点击左上角工具栏上的新建或者重启下QGIS


坐标系统选择：在主界面的右下角有一个叫EPSG:4326的东东，单击它，出现一个对话框，勾选Enable 'on the fly' CRS transformation，然后在Filter中输入3857，选中后，点击ok


**加载省份数据**

点击左上角的 add vector feature按钮，选择我们要加载的中国省份矢量边界数据(我这里以province.shp命名)

在layer panel中选中province图层，右键选择 properties，在打开的对话框中，选择Style——>Simple fill，进行如下设置：

- Fill 设置为 Transparent fill
- border 设置为白色
- Border width 设置为0.1

**加载村庄数据**

与省份的方法类似，加载后效果如下，比较难看 ,所有的点都堆叠在一起了

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/weixin/p3.jpg)

同样，我们右键cun这个图层，选择properties，在Style中选择Categorized     

然后点击右面那个数学符号，在新弹出的对话框中输入

right(name,1)     
这是用来获取每个村庄name的最后一个字，用于后面的筛选。 

点击add添加value为村，Legend也是村，这里可以添加多个，比如寨、屯、湾等等 


点击apply后可以看到，点没之前加载的那么多了，而且cun图层下面多了一个子类 


再进到properties中，双击Symbol下面的那个圆点，将会弹出一个新的对话框，用来设置点的属性

在新打开的对话框中按如下方式设置，然后点击ok

- fill：红色
- outline：transparent border
- size：0.1

这里要注意，并不一定将size设置成0.1，这个要根据点的数量而定，如果点的数量比较少，那么你需要将size调大，显示效果才会好

**注意**      
由于可能数据导入时造成中文乱码问题，需要在设置中设置数据的编码方式  

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/weixin/p6.jpg)

**多种尾字图叠加效果** 

如何生成混在一起的图呢，只需要将所有的尾字都勾选上就可以

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/weixin/p7.jpg)


可以由图得知   

- 村庄分布沿长城与胡焕庸线分布较为明显。
- 村尾字全国分布，新疆，西藏等地较少.
- 庄尾字主要分布在华北等地区，数量较多。
- 其他如屯等主要分布在东北，寨等主要分布在云南等。

**参考链接**    
[中国村庄名称尾字图 教程](https://zhuanlan.zhihu.com/p/22659847)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/weixin/p5.jpg)













