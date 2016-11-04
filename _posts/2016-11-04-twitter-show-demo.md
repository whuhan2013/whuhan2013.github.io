---
layout: post
title: TWitch Streamers实现
date: 2016-11-4
categories: blog
tags: [javascript]
description: TWitch Streamers实现
---

TWitch Streamers主要实现了以下功能       

- 根据api获取twitter好友数据并展示      
- 根据在线与离线状态可以切换效果
- 根据页面大小，布局会自动变化      


![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p5.png)       

**index.html**      

```
<!DOCTYPE html>
<html>

<head>
    <title>Twitter</title>
    <!-- 新 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap.min.css">
    <!-- 可选的Bootstrap主题文件（一般不用引入） -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap-theme.min.css">
    <link rel="stylesheet" type="text/css" href="css/main.css">
</head>

<body>
    <div class="container">
        <div class="row" id="header">
            <h1>Twitch Streamers</h1>
            <div class="menu">
                <div class="selector active" id="all">
                    <div class="circle"></div>
                    <p>All</p>
                </div>
                <div class="selector" id="online">
                    <div class="circle"></div>
                    <p>Online</p>
                </div>
                <div class="selector" id="offline">
                    <div class="circle"></div>
                    <p>Offline</p>
                </div>
            </div>
        </div>
        <div id="display">
        </div>
        <div class="row" id="footer">
        </div>
    </div>
    <!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
    <script src="http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>
    <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
    <script src="http://cdn.bootcss.com/bootstrap/3.3.0/js/bootstrap.min.js"></script>
    <script type="text/javascript" src="js/main.js"></script>
</body>

</html>
```


![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p6.png) 


**main.css**     


```
 @import url(https://fonts.googleapis.com/css?family=Oswald:400,700|Droid+Serif:400,400italic);
body {
  background-color: #ecf0e7;
  font-family: 'Droid Serif', serif;
  font-size: 14px;
  color: #8ea7c2;
}

a, a:focus, a:hover, a:visited {
  color: #b8cca6;
}

h1 {
  font-family: 'Oswald', sans-serif;
  font-weight: bold;
  text-transform: uppercase;
  text-align: left;
  margin: 15px 0px;
  font-size: 3em;
}

.container {
  background-color: #e1e1e6;
  margin: 0px auto;
  padding: 0px;
  max-width: 700px;
}

.row {
  margin: 2px 0px;
  padding: 5px;
  line-height: 50px;
}

.menu {
  position: absolute;
  right: 0px;
  bottom: 5px;
  color: #5c5457;
  font-family: 'Oswald', sans-serif;
  font-size: 0.8em;
  font-weight: bold;
  text-transform: uppercase;
}
.menu #online .circle {
  background-color: #b8cca6;
}
.menu #offline .circle {
  background-color: #8ea7c2;
}
.menu .active, .menu .selector:hover {
  width: 65px !important;
  -webkit-transition: width 0.4s linear;
  transition: width 0.4s linear;
  cursor: pointer;
}
.menu .selector {
  line-height: 20px;
  height: 20px;
  background-color: #e1e1e6;
  padding: 0px 5px;
  margin: 2px 0px;
  width: 20px;
  overflow: hidden;
  float: right;
  clear: right;
  -webkit-transition: width 0.4s linear;
  transition: width 0.4s linear;
}
.menu .selector .circle {
  height: 10px;
  width: 10px;
  border-radius: 50%;
  background-color: #5c5457;
  border: 1px solid #5c5457;
  float: left;
  margin: 5px 5px 5px 0px;
}
.menu .selector p {
  float: right;
  margin: 0px;
}

#header, #footer {
  position: relative;
  background-color: #5c5457;
  color: #e1e1e6;
  padding: 5px 65px 5px 15px;
}

.logo {
  max-width: 50px;
  max-height: 50px;
  border-radius: 50%;
  border: 3px solid #e1e1e6;
}

.offline {
  background-color: #4a5e82;
}

.online {
  background-color: #b8cca6;
  color: #5c5457;
}
.online a, .online a:focus, .online a:hover, .online a:visited {
  color: #4a5e82;
}

#streaming {
  font-style: italic;
}

#name, #streaming {
  line-height: 25px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  text-align: center;
}

@media (min-width: 768px) {
  .container {
    margin: 10px auto;
  }

  #name, #streaming {
    line-height: 50px;
    height: 50px;
  }

  #header {
    padding-left: 65px;
  }
}
```

注意：当界面压缩时，行高会变为原来的一半，正好原先并列的变成两行，与图标对齐


![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p7.png) 


**main.js**     
点击时会将hidden属性添加或移除到控件上，以实现点击的转换

```
var channels = ["ESL_SC2", "OgamingSC2", "cretetion", "freecodecamp", "storbeck", "habathcx", "RobotCaleb", "noobs2ninjas"];

function getChannelInfo() {
    channels.forEach(function(channel) {
        function makeURL(type, name) {
            return 'https://wind-bow.hyperdev.space/twitch-api/' + type + '/' + name + '?callback=?';
        };
        $.getJSON(makeURL("streams", channel), function(data) {
            var game,
                status;
            if (data.stream === null) {
                game = "Offline";
                status = "offline";
            } else if (data.stream === undefined) {
                game = "Account Closed";
                status = "offline";
            } else {
                game = data.stream.game;
                status = "online";
            };
            $.getJSON(makeURL("channels", channel), function(data) {
                var logo = data.logo != null ? data.logo : "https://dummyimage.com/50x50/ecf0e7/5c5457.jpg&text=0x3F",
                    name = data.display_name != null ? data.display_name : channel,
                    description = status === "online" ? ': ' + data.status : "";
                html = '<div class="row ' +
                    status + '"><div class="col-xs-2 col-sm-1" id="icon"><img src="' +
                    logo + '" class="logo"></div><div class="col-xs-10 col-sm-3" id="name"><a href="' +
                    data.url + '" target="_blank">' +
                    name + '</a></div><div class="col-xs-10 col-sm-8" id="streaming">' +
                    game + '<span class="hidden-xs">' +
                    description + '</span></div></div>';
                status === "online" ? $("#display").prepend(html) : $("#display").append(html);
            });
        });
    });
};



$(document).ready(function() {
    getChannelInfo();
    $(".selector").click(function() {
        $(".selector").removeClass("active");
        $(this).addClass("active");
        var status = $(this).attr('id');
        if (status === "all") {
            $(".online, .offline").removeClass("hidden");
        } else if (status === "online") {
            $(".online").removeClass("hidden");
            $(".offline").addClass("hidden");
        } else {
            $(".offline").removeClass("hidden");
            $(".online").addClass("hidden");
        }
    })
});
```


**Source:**[twitterCode](https://github.com/whuhan2013/freeCodeCampProject/tree/master/twitter)        
**LiveDemo:**[Twitter](https://whuhan2013.github.io/web/twitter/index.html)








