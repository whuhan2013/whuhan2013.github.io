---
layout: post
title: 瓦片地图与geoserver发布
date: 2016-10-17
categories: blog
tags: [地图开发]
description: 地图开发
---

**本文主要包括以下内容**   

- TileMill生成Tile影像金字塔（.mbtiles压缩文件）
- Mbutil(https://github.com/mapbox/mbutil)解压缩
- Apache HTTP Server(或tomcat) 建立web瓦片服务
- 客户端调用（ http://www.arcgis.com/home）测试


**首先将数据导入postgres数据库中**  

```
osm2pgsql -s -d simple -U postgres -H localhost -S default.style E:\SzgcSj2016\osmosis-latest\bin\Wuhan.pbf --prefix wuhan -W



SELECT osm_id,edge_id, (dp).path[1] As
index, ST_AsText((dp).geom) As wktnode from
(
select osm_id,row_number() over() as
edge_id,ST_DumpPoints(way) as dp from
wuhan_roads limit 100)x

//将geometry的空间参考改为4326（因为默认为900913）

ALTER TABLE wuhan_roads ALTER
COLUMN way TYPE
geometry(LineString,4326) USING
ST_Transform(way,4326);

ALTER TABLE wuhan_line ALTER
COLUMN way TYPE
geometry(LineString,4326) USING
ST_Transform(way,4326);

ALTER TABLE wuhan_point ALTER
COLUMN way TYPE geometry(Point,4326)
USING ST_Transform(way,4326);

//给表格wuhan_polygon添加属性way2 geometry

ALTER TABLE wuhan_polygon ADD COLUMN way2

geometry;



//给表格wuhan_polygon的属性way2 geometry添加格式：(way,4326)

update wuhan_polygon SET way2=ST_Transform(way,4326);



//删除表格wuhan_polygon的属性way

ALTER TABLE wuhan_polygon DROP COLUMN way;



//给表格wuhan_polygon的属性way2改民为way

ALTER TABLE wuhan_polygon RENAME way2 TO way;
```

**TileMill配置postgres数据连结**

- 新建项目
- 添加图层
- 设置工程的默认渲染方式
- 样式配置
- 条件性样式
- 工具提示
- 图例
- 输出地图

连接后根据ID写样式配置  

```
#wuhanpolygon {
  ::outline {
    line-color: #3c22d6;
    line-width: 2;
    line-join: round;
  }
  polygon-fill: #03b6ad;
}

host=localhost port=5432 user=postgres password=root dbname=simple

#wuhanroads {
  ::outline {
    line-color: #3c22d6;
    line-width: 2;
    line-join: round;
  }
  polygon-fill: #03b6ad;
}


#wuhanpoint {
  marker-width:6;
  marker-fill:#8b9091; //f45
  marker-line-color:#813;
  marker-allow-overlap:true;

}
```

然后导出mbtils文件，用python或者QGis安装插件解压即可，所得即瓦片地图，把文件夹放在tomact中root文件夹下即可访问

```
<table>
  <td>
  <tr ><img src="http://localhost:8080/wuhan/13/6698/4831" ></tr>
  <tr><img src="http://localhost:8080/wuhan/13/6699/4831"> </tr>
  </td>
  <td>
  <tr><img src="http://localhost:8080/wuhan/13/6698/4832" ></tr>
  <tr><img src="http://localhost:8080/wuhan/13/6699/4832"> </tr>
  </td>
  </table>
````


在tomcat中webapp下导入geoserver,启动geoserver,连接postgres，分别发布wuhan_roads,wuhan_point,wuhan_line,wuhan_polygen等表，可以在浏览器中查看效果如下：


![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/map/p2.jpg)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/map/p3.jpg)



