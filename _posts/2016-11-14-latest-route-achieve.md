---
layout: post
title: 最短路径实现
date: 2016-11-14
categories: blog
tags: [地图开发]
description: 最短路径实现
---

**主要工具**           

1. QGIS建立拓扑关系        
2. Postgres存储数据表         
3. Geoserver发布相关服务           


#### QGIS建立拓扑关系     
使用v.clean运行，并用DBManager即可以建立拓扑关系并导入数据库。       

**注意**       
QGIS2.16有数据溢出问题，使用QGIS2.14可以解决这个问题                

#### Postgres存储数据表         

导入wuhan_road_topo后可以进行相应处理        

```
drop table if exists tmp_pts_topo;
create table tmp_pts_topo
as
select 
osm_id
,(st_dump(geom)).geom as geom
,st_startpoint((st_dump(geom)).geom)
,st_endpoint((st_dump(geom)).geom)
,st_length((st_dump(geom)).geom)*110 as dis
from wuhan_road_topo;

alter table tmp_pts_topo add column id serial;
select * from tmp_pts_topo limit 10;

drop table if exists tmp_pts;
create table tmp_pts(geom geometry);

insert into tmp_pts
select geom from
(
select st_startpoint as geom from tmp_pts_topo 
union all 
select st_endpoint  as geom from tmp_pts_topo
)x
group by geom;


alter table tmp_pts add column id serial; 

select * from tmp_pts limit 10;
select * from tmp_pts_topo limit 10;
alter table tmp_pts_topo add column start_point int;
alter table tmp_pts_topo add column end_point int;

create index indx_geom_tmp_pts on tmp_pts 
using gist(geom);

create index indx_geom_tmp_pts_1 on tmp_pts_topo 
using gist(st_startpoint);

create index indx_geom_tmp_pts_2 on tmp_pts_topo 
using gist(st_endpoint);


select * from tmp_pts limit 10;
select * from tmp_pts_topo limit 10;

update tmp_pts_topo t1 set start_point=t2.id
from tmp_pts t2
where st_dwithin(t1.st_startpoint,t2.geom,0);

update tmp_pts_topo t1 set end_point=t2.id
from tmp_pts t2
where st_dwithin(t1.st_endpoint,t2.geom,0);

insert into tmp_pts(geom)
select st_startpoint 
from tmp_pts_topo 
where start_point is null
union
select st_endpoint 
from tmp_pts_topo 
where end_point is null;


update tmp_pts_topo t1 set start_point=t2.id
from tmp_pts t2
where st_dwithin(t1.st_startpoint,t2.geom,0)
and start_point is null;

update tmp_pts_topo t1 set end_point=t2.id
from tmp_pts t2
where st_dwithin(t1.st_endpoint,t2.geom,0)
and end_point is null;

delete from tmp_pts_topo where id in
(
select id from(
select id,row_number() 
over(partition by start_point,end_point order by dis) 
from tmp_pts_topo
where (start_point,end_point) in
(
select start_point,end_point
from tmp_pts_topo 
group by start_point,end_point
having count(*)>1
)
)x
where row_number>1
);

select *
from tmp_pts_topo 
where start_point=1138 and end_point=1143;

select count(*) from tmp_pts_topo;

select * from tmp_pts_topo limit 1;

select * from tmp_pts_topo where start_point=10997;

delete from tmp_pts_topo where start_point is null or end_point is null;
```

**注意**         
要删除空白点，不然无法正常运行             


**测试查询**         
选定一个起点与终点，测试有没有结果        

```
with start_point as
(
select st_setsrid(
st_makepoint(114.25,30.57)
,4326) as p
)
,end_point as
(
select st_setsrid(st_makepoint(114.23,30.56),4326) as p
)
select t2.seq,st_asgeojson(t1.geom) from 
(
SELECT seq, 
id1 AS node, 
id2 AS edge, 
cost FROM pgr_dijkstra('
SELECT id,                        
start_point as source,                        
end_point as target,                        
dis AS cost                       
FROM tmp_pts_topo',                
(
select id from tmp_pts
where st_dwithin(
(select p from start_point)
,geom,0.05)
order by st_distance(
(select p from start_point)
,geom)
limit 1
)
, 
(
select id from tmp_pts
where st_dwithin(
(select p from end_point)
,geom,0.05)
order by st_distance(
(select p from end_point)
,geom)
limit 1
), false, false)
)t2 left join tmp_pts_topo t1
on t1.id=t2.edge
where t2.edge>0
order by t2.seq;
```

#### Geoserver发布相关服务       
使用Geoserver连接postgres并发布wuhan_road_topo，并配置新的SQL视图，SQL视图可以接收参数返回相应的视图     

可参见：[GeoServer的SQL Views详解](http://blog.csdn.net/freeland1/article/details/49737793)  

将SQL语句修改为      

```
with start_point as
(
select st_setsrid(
st_makepoint(%startLon%,%startLat%)
,4326) as p
)
,end_point as
(
select st_setsrid(st_makepoint(%endLon%,%endLat%),4326) as p
)
select t2.seq,(t1.geom) from 
(
SELECT seq, 
id1 AS node, 
id2 AS edge, 
cost FROM pgr_dijkstra('
SELECT id,                        
start_point as source,                        
end_point as target,                        
dis AS cost                       
FROM tmp_pts_topo',                
(
select id from tmp_pts
where st_dwithin(
(select p from start_point)
,geom,0.05)
order by st_distance(
(select p from start_point)
,geom)
limit 1
)
, 
(
select id from tmp_pts
where st_dwithin(
(select p from end_point)
,geom,0.05)
order by st_distance(
(select p from end_point)
,geom)
limit 1
), false, false)
)t2 left join tmp_pts_topo t1
on t1.id=t2.edge
where t2.edge>0
order by t2.seq
```

发布成功后可以通过修改参数，如http://localhost:8080/geoserver/wuhanwork/wms?service=WMS&version=1.1.0&request=GetMap&view&layers=wuhan:tt&styles=&bbox=114.22769686949,30.559729927104,114.246243846172,30.5719569775606&width=512&height=337&srs=EPSG:4326&format=application/openlayers&viewparams=startLon:114.25;startLat:30.57;endLon:114.39;endLat:30.70 来访问         
即可通过输入起点与终点，返回最短路径   

![](https://github.com/whuhan2013/myImage/blob/master/R/p6.png?raw=true)

