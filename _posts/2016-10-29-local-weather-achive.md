---
layout: post
title: LocalWeather实现
date: 2016-10-29
categories: blog
tags: [javascript]
description: LocalWeather实现
---



本项目主要实现了根据ip获得地址当地天气，并根据天气显示不同图片，同时可以切换摄氏与华氏温度


**在垂直方向上居中实现**

在垂直方向上居中利用display:table实现

```
.site-wrapper {
  display: table;
  width: 100%;
  height: 100%; /* For at least Firefox */
  min-height: 100%;
  -webkit-box-shadow: inset 0 0 100px rgba(0,0,0,.5);
          box-shadow: inset 0 0 100px rgba(0,0,0,.5);
}

.site-wrapper-inner {
  display: table-cell;
  
  vertical-align: middle;
}
```


**html界面**     

```
<!DOCTYPE html>
<html>

<head>
    <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests" />
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <link rel="stylesheet" type="text/css" href="css/main.css" />
    <link rel="icon" href="images/rainbow.ico" />
    <title>Local Weather</title>
</head>

<body>
    <div class="site-wrapper">
        <div class="site-wrapper-inner">
            <div class="container">
                <p class="cover-heading information"></p>
                <p class="lead">
                    <a class="btn btn-lg btn-default btn-check">Check Weather</a>
                    <a class="btn btn-lg btn-default btn-celsius">Convert to Celsius</a>
                    <a class="btn btn-lg btn-default btn-fahrenheit">Convert to Fahrenheit</a>
                </p>
            </div>
        </div>
        <div class="foot">
          <p>Local Weather for <a target="_blank" href="http://codepen.io/whuhan2013/full/mrZdGK/">FreeCodeCamp</a> by <a target="_blank" href="http://whuhan2013.github.io/">Ricarod.M.Jiang</a>.</p>
        </div>
    </div>
    <div class="test"></div>
    <div class="test2"></div>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
    <script src="js/weather.js"></script>
</body>

</html>
```


**css页面** 

```
.btn-celsius, 
.btn-fahrenheit {
  display: none;
}

a,
a:focus,
a:hover {
  color: #fff;
}

.btn-default,
.btn-default:hover,
.btn-default:focus {
  color: #333;
  text-shadow: none;
  background-color: #fff;
  border: 1px solid #fff;
}

.btn-lg {
  padding: 10px 20px;
  font-weight: bold;
}

.information {
  font-size: 36px;
  font-weight: 600;
}

html,
body {
  height: 100%;
  background-size: cover;
  
}

body {
  color: #fff;
  text-align: center;
  text-shadow: 0 1px 3px rgba(0,0,0,.5);

}

.site-wrapper {
  display: table;
  width: 100%;
  height: 100%; /* For at least Firefox */
  min-height: 100%;
  -webkit-box-shadow: inset 0 0 100px rgba(0,0,0,.5);
          box-shadow: inset 0 0 100px rgba(0,0,0,.5);
  background-color: #333;
}

.test{
  width: 100%;
  height: 100%;
  background-color: #FFF;
}

.test2{
   width: 100%;
  height: 100%;
  background-color: #333;
}

.site-wrapper-inner {
  display: table-cell;
  
  vertical-align: middle;
  width: 100%;
  
}

.container {
  margin-right: auto;
  margin-left: auto;
  width: 700px;
  padding: 0 20px;
}


.foot{
  display: table-row;  
  height: 10%;
  vertical-align: center;
  color: rgba(255,255,255,.5);

}
```

**通过jquery获得天气数据**     
根据结果更换图片，按钮形式，显示内容  

```
$(document).ready(function() {
  $.getJSON('http://ip-api.com/json', function(ipAddress) {
    $.getJSON('http://api.openweathermap.org/data/2.5/weather?lat=' + ipAddress.lat + '&lon=' + ipAddress.lon + '&appid=3695e5e886e4a1b016cf201000ec807e', function(forecast) {
      var celsius = forecast.main.temp - 273.15;
      var fahrenheit = celsius * 1.8 + 32;
      var backgroundPic = forecast.weather[0].icon.substring(0, 2);

      var $body = $('.site-wrapper');
      if (backgroundPic === '01' || backgroundPic === '02' || backgroundPic === '03') {
        $body.css('background-image', 'url("images/clear.jpg")');
      } else if (backgroundPic === '04') {
        $body.css('background-image', 'url("images/broken.jpg")');
      } else if (backgroundPic === '09') {
        $body.css('background-image', 'url("images/drizzle.jpg")');
      } else if (backgroundPic === '10') {
        $body.css('background-image', 'url("images/rain.jpg")');
      } else if (backgroundPic === '11') {
        $body.css('background-image', 'url("images/thunder.jpg")');
      } else if (backgroundPic === '13') {
        $body.css('background-image', 'url("images/snow.jpg")');
      } else if (backgroundPic === '50') {
        $body.css('background-image', 'url("images/fog.jpg")');
      } else {
        $body.css('background-image', 'url("images/def.jpg")');
      }
      $('.information').text('Hello ' + ipAddress.city + ' from Seattle.');
      $('.btn-check').on('click', function() {
        $('.btn-check').hide();
        $('.btn-celsius').show();
        $('.information').text('The current temperature in ' + ipAddress.city + ' is ' + fahrenheit.toFixed(0) + ' degrees Fahrenheit.');
      });
      $('.btn-celsius').on('click', function() {
        $('.btn-celsius').hide();
        $('.btn-fahrenheit').show();
        $('.information').text('The current temperature in ' + ipAddress.city + ' is ' + celsius.toFixed(0) + ' degrees Celsius.');
      });
      $('.btn-fahrenheit').on('click', function() {
        $('.btn-fahrenheit').hide();
        $('.btn-celsius').show();
        $('.information').text('The current temperature in ' + ipAddress.city + ' is ' + fahrenheit.toFixed(0) + ' degrees Fahrenheit.');
      });
    });
  });
});
```

**问题**       

1. This request has been blocked; the content must be served over HTTPS.        
当把网页放到https目录下时，似乎不能请求http资源，导致网页不能使用，但这个接口又似乎没有https版本，暂时无法解决。

**源码:**[SourceCode](https://github.com/whuhan2013/freeCodeCampProject/tree/master/weather)   
**liveDemo:**[Weather](https://whuhan2013.github.io/web/weather/index.html)    

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/javascript/p13.png)


