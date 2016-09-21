---
layout: post
title: postgres与osm初步使用
date: 2016-9-20
categories: blog
tags: [地图开发]
description: 地图开发
---



**本文包括以下内容**   

- postgreSQL数据库，用来存放地图原始数据
- osm2pgsql 用来将osm地图数据导入到postgreSQL

**OSM数据**

OpenStreetMap（简称OSM）是一个网上地图众筹（crowd sourcing）项目，目标是创造一个内容自由且能让所有人编辑的世界地图


**osm数据特点**

- 数据来源多样，海量数据但数据质量参差不齐（错误、不一致），需要大量的后续数据处理
- 按照数据的类别，用不同的标签标示数据类别
- 将数据进行汇编，按进行全球（Planet OSM, OSM）或区域发布
- 文件大：XML variant over 600+GB uncompressed, 50+ GB bz2 compressed and 30+GB for PBF

OpenStreetMap包括空间数据以及属性数据。其中空间数据主要包括三种：点（Nodes）、路（Ways）和关系（Relations），这三种原始构成了整个地图画面。其中，Nodes定义了空间中点的位置；Ways定义了线或区域；Relations（可选的）定义了元素间的关系, 属性数据Tags用于描述上述矢量数据基元

node通过经纬度定义了一个地理坐标点。同时，还可以height=*标示物体所海拔；通过layer=* 和 level=*，可以标示物体所在的地图层面与所在建筑物内的层数；通过place=* and name=*来表示对象的名称。同时，way也是通过多个点（node）连接成线（面）来构成的。


通过2-2000个点（nodes）构成了way。way可表示如下3种图形事物（非闭合线（Open polyline ）、闭合线（Closed polyline）、区域（Area ））。对于超过2000 nodes的way，可以通过分割来处理。

**OSM分类Features和标签（Tag）**

各feature都通过tag来记录数据信息,通过‘key’ and a ‘value’来对数据进行记录。例如，可以通过highway=residential来定义居住区道路；同时，可以使用附加的命名空间来添加附加信息，例如：maxspeed:winter=*就表示冬天的最高限速


**postgressql数据库安装**

[开源免费数据库：PostgreSQL下载安装教程_百度经验](http://jingyan.baidu.com/article/fa4125acb0ae8328ac709292.html)



#### OSM数据提取

下载osm：http://dev.openstreetmap.org/~bretth/osmosis-build/osmosis-latest.zip





**下载中国地图数据**                     
http://download.geofabrik.de/asia/china.html


**数据裁剪**

区域裁减,把中国地图裁剪成武汉地图：

```
osmosis --read-pbf file="china-latest.osm.pbf" --used-node --bounding-box left=113.9502 right=114.4762 top=30.7643 bottom=30.4291 clipIncompleteEntities="true" --write-pbf Wuhan.pbf
```

提取武汉地图的高速公路数据

```
osmosis --read-pbf file="wuhan.pbf" --tf accept-ways highway=* clipIncompleteEntities="true" --write-pbf Wuhan_highway.pbf
```

**将数据导入postgres数据库**

- create extension hstore;
- create extension postgis;
- create extension pgrouting;
- 运行数据库ddl： pgsnapshot_schema_0.6.sql
- 导入数据：osmosis --read-pbf file="wuhan_highway.pbf" --wp host=localhost database=osm user=postgres password=postgres

1. 首先在postgres中添加3个扩展             
2. 再运行osmosis-latest\script文件夹下的 pgsnapshot_schema_0.6.sql脚本
3. 运行导入指令即可


**安装QGIS并连接postgres可以在地图上看到导入的数据**

![](https://github.com/whuhan2013/ImageRepertory/blob/master/map/p1.png?raw=true)