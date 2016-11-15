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

**客户端实现**       

```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=Edge">
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no, width=device-width">
    <!-- 新 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap.min.css">
    <!-- 可选的Bootstrap主题文件（一般不用引入） -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap-theme.min.css">
    <link rel="stylesheet" href="./css/prism.css" type="text/css">
    <link rel="stylesheet" href="./css/ol.css" type="text/css">
    <link rel="stylesheet" href="./css/layout.css" type="text/css">
    <link rel="stylesheet" href="./css/popup.css">
    <link rel="stylesheet" type="text/css" href="./css/main.css">
    <script src="js/ol-debug.js"></script>
    <script src="js/ol-deps.js"></script>
    <script src="js/ol.js"></script>
    <title>HomeWork</title>
</head>

<body>
    <div class="mycontainer">
        <div class="row" id="header">
            <h1>Digital engineering practice</h1>
        </div>
    </div>
    <div class="container-fluid">
        <div class="row-fluid">
            <div class="span12">
                <div id="map" class="map"></div>
                <div id="popup" class="ol-popup">
                    <a href="#" id="popup-closer" class="ol-popup-closer"></a>
                    <div id="popup-content"></div>
                </div>
                <!--鼠标点击-->
                <div class="buttonlay">
                    <button type="button" id="btStart" onclick="btStart_click()" data-original-title="Click" class="btn btn-lg btn-default btn-check mybutton"><span>Click to input start point</span></button>
                    <button type="button" id="btEnd" onclick="btEnd_click()" data-original-title="Click" class="btn btn-lg btn-default btn-check mybutton"><span>Click to input end point</span> </button>
                    <button type="button" id="selectShort" onclick="select_click()" data-original-title="Click" class="btn btn-lg btn-default btn-check mybutton"> <span>Click to slect shortest way</span></button>
                </div>
            </div>
        </div>
    </div>
</body>
<script type="text/javascript">
/**
 * Elements that make up the popup.
 */
var container = document.getElementById('popup');
var content = document.getElementById('popup-content');
var closer = document.getElementById('popup-closer');

//记录按钮动作
var ableSt = false;
var ableEd = false;

//起点经度，起点维度，终点经度，终点纬度
var st_lon, st_lat, ed_lon, ed_lat;


/**
 * Create an overlay to anchor the popup to the map.
 */
var overlay = new ol.Overlay( /** @type {olx.OverlayOptions} */ ({
    element: container,
    autoPan: true,
    autoPanAnimation: {
        duration: 250
    }
}));

/**
 * 加载瓦片图层
 **/
var source = new ol.source.XYZ({
    url: 'http://localhost:8080/tilemill/{z}/{x}/{y}.png'
});

var center = ol.proj.transform([114.2207, 30.5960], 'EPSG:4326', 'EPSG:3857');
var map = new ol.Map({
    logo: false,
    layers: [
        new ol.layer.Tile({
            source: source
        })
    ],
    overlays: [overlay],
    target: 'map',
    view: new ol.View({
        maxZoom: 18,
        center: center,
        zoom: 10
    })
});



/**
 * Add a click handler to hide the popup.
 * @return {boolean} Don't follow the href.
 */
closer.onclick = function() {
    overlay.setPosition(undefined);
    closer.blur();
    return false;
};


/**
 * Add a click handler to the map to render the popup.
 */
map.on('singleclick', function(evt) {
    var coordinate = evt.coordinate;
    var hdms = ol.coordinate.toStringHDMS(ol.proj.transform(
        coordinate, 'EPSG:3857', 'EPSG:4326'));
    var xy = ol.proj.transform(coordinate, 'EPSG:3857', 'EPSG:4326').toString();
    var lon_string = xy.split(",")[0];
    var lat_string = xy.split(",")[1];

    var lon_number = new Number(lon_string);
    var lat_number = new Number(lat_string);

    if (ableSt) {
        st_lon = lon_number.toFixed(4);
        st_lat = lat_number.toFixed(4);
        content.innerHTML = '<p>You put start point here:</p><code>' + hdms + '</code>';
        ableSt = false;
    } else {
        if (ableEd) {
            ed_lon = lon_number.toFixed(4);
            ed_lat = lat_number.toFixed(4);
            content.innerHTML = '<p>You put end point here:</p><code>' + hdms + '</code>';
            ableEd = false;
        } else {
            content.innerHTML = '<p>You clicked here:</p><code>' + hdms + '</code>';
        }
    }

    overlay.setPosition(coordinate);
});

/**
 *按钮相应事件
 **/
function btStart_click() {
    ableSt = true;
}

function btEnd_click() {
    ableEd = true;
}

//最短路径按钮
function select_click() {
    overlay.setPosition(undefined);
    closer.blur();

    var getwayurl = 'http://localhost:8080/geoserver/wuhan/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=wuhan:wuhan&maxFeatures=50&outputFormat=application/json&viewparams=' + 'st_lon:' + st_lon + ';st_lat:' + st_lat + ';ed_lon:' + ed_lon + ';ed_lat:' + ed_lat;
    var getMywayurl = 'http://localhost:8080/geoserver/wuhanwork/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=wuhanwork:tt&maxFeatures=50&outputFormat=application/json&viewparams=' + 'startLon:' + st_lon + ';startLat:' + st_lat + ';endLon:' + ed_lon + ';endLat:' + ed_lat;

    var roadsource = new ol.layer.Vector({
        source: new ol.source.Vector({       
            url: getMywayurl,
            format: new ol.format.GeoJSON({
                extractStyles: true
            }),
            style: new ol.style.Style({
                stroke: new ol.style.Stroke({
                    color: [255, 255, 255, 1],
                    width: 30
                })
            })
        })
    });

    map.addLayer(roadsource);
}
</script>

</html>
```

**源代码:**[SourceCode](https://github.com/whuhan2013/mywebproject/tree/master/DigitalEnginn)

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/R/p7.png)