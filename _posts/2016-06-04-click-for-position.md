---
layout: post
title: 百度地图知识点总结
date: 2016-6-4
categories: blog
tags: [地图开发]
description: 百度地图点击得到地址
---

### 本文主要包括以下内容 

1. 百度地图通过点击得到地址
2. android studio获取sha1

### 百度地图通过点击得到地址

地理编码指的是将地址信息建立空间坐标关系的过程。有可分为正向地图编码和反向地图编码。
正向地理编码指的是由地址信息转换为坐标点的过程，反向编码则将经纬度转化为地址。  


```
private BaiduMap bdMap;
 bdMap.setOnMapClickListener(new BaiduMap.OnMapClickListener() {
            @Override
            public void onMapClick(LatLng latLng) {
                //Toast.makeText(getApplication(),latLng.longitude+","+latLng.latitude,Toast.LENGTH_SHORT).show();
                initGEO(latLng);
            }

            @Override
            public boolean onMapPoiClick(MapPoi mapPoi) {
                Toast.makeText(getApplication(),mapPoi.getName(),Toast.LENGTH_SHORT).show();
                return false;
            }
        });
```


```
public void initGEO(LatLng latLng)
    {
        GeoCoder mSearch=GeoCoder.newInstance();

        OnGetGeoCoderResultListener listener = new OnGetGeoCoderResultListener() {
            public void onGetGeoCodeResult(GeoCodeResult result) {
                if (result == null || result.error != SearchResult.ERRORNO.NO_ERROR) {
                    //没有检索到结果
                }
                //获取地理编码结果

            }

            @Override
            public void onGetReverseGeoCodeResult(ReverseGeoCodeResult result) {
                if (result == null || result.error != SearchResult.ERRORNO.NO_ERROR) {
                    //没有找到检索结果
                }
                //获取反向地理编码结果
                Toast.makeText(getApplication(),result.getAddress(),Toast.LENGTH_SHORT).show();
            }
        };
        ReverseGeoCodeOption reverseGeoCodeOption=new ReverseGeoCodeOption();

        mSearch.setOnGetGeoCodeResultListener(listener);
        mSearch.reverseGeoCode(reverseGeoCodeOption.location(latLng));
    }

```


### 参考链接

[百度地图开发中怎么实现点击地图中的字或建筑获取到位置，如图_百度知道](http://zhidao.baidu.com/link?url=JhK9x9lzhxr6CK0owZRRuPZj_r4ey9yjD9-EaMr636A9eILYeW6HIAfGiKTzAaJ1gEOBYu4cnrB61HKY2pDJVmHAIIoOguDk-8Nbm00OyA3)

[androidsdk/guide/retrieval - Wiki](http://lbsyun.baidu.com/index.php?title=androidsdk/guide/retrieval)




### android studio获取sha1

在CMD中输入keytool -list -v -keystore debug.keystore，

详情见：

[AndroidAPI申请密钥 - GeoLocation](http://developer.baidu.com/map/geosdk-android-key.htm)


### 注意

要找到正确的debug.keystore，由Android_SDK_HOME在环境变量中配置的路径为准。




