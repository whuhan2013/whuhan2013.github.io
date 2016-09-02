---
layout: post
title: 高德API+Python解决租房问题
date: 2016-9-2
categories: blog
tags: [网络爬虫]
description: 爬虫58 
---

**概述**        
本文主要实现了以下功能         

1. python爬取58同城房源数据       
2. 利用高德地图API将房源数据标示在地图上
3. 上传到服务器并使用        


### 爬取数据  

```
#-*- coding:utf-8 -*-
from bs4 import BeautifulSoup
from urlparse import urljoin
import requests
import csv

url = "http://bj.58.com/pinpaigongyu/pn/{page}/?minprice=2000_4000"

#已完成的页数序号，初时为0
page = 0

csv_file = open("rent.csv","wb") 
csv_writer = csv.writer(csv_file, delimiter=',')

while True:
    page += 1
    print "fetch: ", url.format(page=page)
    response = requests.get(url.format(page=page))
    html = BeautifulSoup(response.text)
    house_list = html.select(".list > li")

    # 循环在读不到新的房源时结束
    if not house_list:
        break

    for house in house_list:
        house_title = house.select("h2")[0].string.encode("utf8")
        house_url = urljoin(url, house.select("a")[0]["href"])
        house_info_list = house_title.split()

        # 如果第二列是公寓名则取第一列作为地址
        if "公寓" in house_info_list[1] or "青年社区" in house_info_list[1]:
            house_location = house_info_list[0]
        else:
            house_location = house_info_list[1]

        house_money = house.select(".money")[0].select("b")[0].string.encode("utf8")
        csv_writer.writerow([house_title, house_location, house_money, house_url])

csv_file.close()
```

**注意**      

python2中用wb的方式打工文件，python3中用w的方式即可，这里牵扯到bytes类型与string类型的转化等因素。

```
bytes解码会得到str
str编码会变成bytes

>>> b'123'.decode('ascii')
'123'
>>> '123'.encode('ascii')
b'123'
```


![](https://dn-anything-about-doc.qbox.me/document-uid8834labid1978timestamp1470289755155.png/wm)



#### 调用高德API展示数据    

**主要实现了以下功能**    

1. 地名自动补全   
2. 路径导航         
3. 显示公交到达圈
4. 信息窗体的使用

```
<html>

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no, width=device-width">
    <title>毕业生租房</title>
    <link rel="stylesheet" href="http://cache.amap.com/lbs/static/main1119.css" />
    <link rel="stylesheet" href="http://cache.amap.com/lbs/static/jquery.range.css" />
    <script src="http://cache.amap.com/lbs/static/jquery-1.9.1.js"></script>
    <script src="http://cache.amap.com/lbs/static/es5.min.js"></script>
    <script src="http://webapi.amap.com/maps?v=1.3&key=22d3816e107f199992666d6412fa0691&plugin=AMap.ArrivalRange,AMap.Scale,AMap.Geocoder,AMap.Transfer,AMap.Autocomplete"></script>
    <script src="http://cache.amap.com/lbs/static/jquery.range.js"></script>
    <style>
    .control-panel {
        position: absolute;
        top: 30px;
        right: 20px;
    }
    
    .control-entry {
        width: 280px;
        background-color: rgba(119, 136, 153, 0.8);
        font-family: fantasy, sans-serif;
        text-align: left;
        color: white;
        overflow: auto;
        padding: 10px;
        margin-bottom: 10px;
    }
    
    .control-input {
        margin-left: 120px;
    }
    
    .control-input input[type="text"] {
        width: 160px;
    }
    
    .control-panel label {
        float: left;
        width: 120px;
    }
    
    #transfer-panel {
        position: absolute;
        background-color: white;
        max-height: 80%;
        overflow-y: auto;
        top: 30px;
        left: 20px;
        width: 250px;
    }
    </style>
</head>

<body>
    <div id="container"></div>
    <div class="control-panel">
        <div class="control-entry">
            <label>选择工作地点：</label>
            <div class="control-input">
                <input id="work-location" type="text">
            </div>
        </div>
        <div class="control-entry">
            <label>选择通勤方式：</label>
            <div class="control-input">
                <input type="radio" name="vehicle" value="SUBWAY,BUS" onClick="takeBus(this)" checked/> 公交+地铁
                <input type="radio" name="vehicle" value="SUBWAY" onClick="takeSubway(this)" /> 地铁
            </div>
        </div>
        <div class="control-entry">
            <label>导入房源：</label>
            <div class="control-input">
                <Button onclick="loadRentLocationByFile('rent2.csv')" >导入</Button>
            </div>
        </div>
    </div>
    <div id="transfer-panel"></div>
    <script>
    var map = new AMap.Map("container", {
        resizeEnable: true,
        zoomEnable: true,
        center: [114.374653,30.542735],
        zoom: 11
    });

    var scale = new AMap.Scale();
    map.addControl(scale);

    var arrivalRange = new AMap.ArrivalRange();
    var x, y, t, vehicle = "SUBWAY,BUS";
    var workAddress, workMarker;
    var rentMarkerArray = [];
    var polygonArray = [];
    var amapTransfer;

    var infoWindow = new AMap.InfoWindow({
        offset: new AMap.Pixel(0, -30)
    });

    var auto = new AMap.Autocomplete({
        input: "work-location"
    });
    
    AMap.event.addListener(auto, "select", workLocationSelected);




    function takeBus(radio) {
        vehicle = radio.value;
        loadWorkLocation()
    }

    function takeSubway(radio) {
        vehicle = radio.value;
        loadWorkLocation()
    }

    function importRentInfo(fileInfo) {
        var file = fileInfo.files[0].name;
        loadRentLocationByFile(file);
    }

    function workLocationSelected(e) {
        workAddress = e.poi.name;
        loadWorkLocation();
    }

    function loadWorkMarker(x, y, locationName) {
        workMarker = new AMap.Marker({
            map: map,
            title: locationName,
            icon: 'http://webapi.amap.com/theme/v1.3/markers/n/mark_r.png',
            position: [x, y]

        });
    }


    function loadWorkRange(x, y, t, color, v) {
        arrivalRange.search([x, y], t, function(status, result) {
            if (result.bounds) {
                for (var i = 0; i < result.bounds.length; i++) {
                    var polygon = new AMap.Polygon({
                        map: map,
                        fillColor: color,
                        fillOpacity: "0.4",
                        strokeColor: color,
                        strokeOpacity: "0.8",
                        strokeWeight: 1
                    });
                    polygon.setPath(result.bounds[i]);
                    polygonArray.push(polygon);
                }
            }
        }, {
            policy: v
        });
    }

    function addMarkerByAddress(address) {
        var geocoder = new AMap.Geocoder({
            city: "武汉",
            radius: 1000
        });
        geocoder.getLocation(address, function(status, result) {
            if (status === "complete" && result.info === 'OK') {
                var geocode = result.geocodes[0];
                rentMarker = new AMap.Marker({
                    map: map,
                    title: address,
                    icon: 'http://webapi.amap.com/theme/v1.3/markers/n/mark_b.png',
                    position: [geocode.location.getLng(), geocode.location.getLat()]
                });
                rentMarkerArray.push(rentMarker);

                rentMarker.content = "<div>房源：<a target = '_blank' href='http://wh.58.com/pinpaigongyu/?key=" + address + "'>" + address + "</a><div>"
                rentMarker.on('click', function(e) {
                    infoWindow.setContent(e.target.content);
                    infoWindow.open(map, e.target.getPosition());
                    if (amapTransfer) amapTransfer.clear();
                    amapTransfer = new AMap.Transfer({
                        map: map,
                        policy: AMap.TransferPolicy.LEAST_TIME,
                        city: "武汉市",
                        panel: 'transfer-panel'
                    });
                    amapTransfer.search([{
                        keyword: workAddress
                    }, {
                        keyword: address
                    }], function(status, result) {})
                });
            }
        })
    }

    function delWorkLocation() {
        if (polygonArray) map.remove(polygonArray);
        if (workMarker) map.remove(workMarker);
        polygonArray = [];
    }

    function delRentLocation() {
        if (rentMarkerArray) map.remove(rentMarkerArray);
        rentMarkerArray = [];
    }

    function loadWorkLocation() {
        delWorkLocation();
        var geocoder = new AMap.Geocoder({
            city: "武汉",
            radius: 1000
        });

        geocoder.getLocation(workAddress, function(status, result) {
            if (status === "complete" && result.info === 'OK') {
                var geocode = result.geocodes[0];
                x = geocode.location.getLng();
                y = geocode.location.getLat();
                loadWorkMarker(x, y);
                loadWorkRange(x, y, 60, "#3f67a5", vehicle);
                map.setZoomAndCenter(12, [x, y]);
            }
        })
    }

    function loadRentLocationByFile(fileName) {
        delRentLocation();
        var rent_locations = new Set();
        $.get(fileName, function(data) {
            data = data.split("\n");
            data.forEach(function(item, index) {
                rent_locations.add(item.split(",")[1]);
            });
            rent_locations.forEach(function(element, index) {
                addMarkerByAddress(element);
            });
        });
    }
    </script>
</body>

</html>
```


### 上传到服务器并运行   

输入python -m SimpleHTTPServer 3000打开服务器，浏览器访问http://119.29.152.242:3000/查看效果：


**参考链接：**[高德API+Python解决租房问题](https://www.shiyanlou.com/courses/599/labs/1978/document)

**源码地址**     

[rent project source code](https://github.com/whuhan2013/pythoncode/tree/master/rent_proj)


**效果如下**            

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/python/p3.png)