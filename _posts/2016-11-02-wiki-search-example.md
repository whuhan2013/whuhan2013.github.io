---
layout: post
title: WIKISearch实现
date: 2016-11-2
categories: blog
tags: [javascript]
description: WIKISearch实现
---

WIKISearch主要实现了以下功能       

- 利用bootstrap实现布局并适配不同界面      
- 利用anguarJS实现controller
- 刚开始时搜索结果为空不显示，开始搜索后界面不在牌中心      


**实现处于屏幕中央的另一个方法**

```
.myvertical {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p3.png)


**index.html**     

```
<!DOCTYPE html>
<html ng-app="WikipediaApplication">

<head>
    <title>WIKI Search</title>
    <!-- 新 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap.min.css">
    <!-- 可选的Bootstrap主题文件（一般不用引入） -->
    <link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap-theme.min.css">
    <!-- jQuery文件。务必在bootstrap.min.js 之前引入 -->
    <script src="http://cdn.bootcss.com/jquery/1.11.1/jquery.min.js"></script>
    <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
    <script src="http://cdn.bootcss.com/bootstrap/3.3.0/js/bootstrap.min.js"></script>
    <!-- Google Font -->
    <link href="https://fonts.googleapis.com/css?family=Raleway:100,300" rel="stylesheet" type="text/css">
    <!-- Custom CSS -->
    <link rel="stylesheet" type="text/css" href="css/main.css">
    <!-- Angular JS -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.min.js"></script>
    <!-- Custom JS -->
    <script type="text/javascript" src="js/app.js"></script>
</head>

<body ng-controller="WikipediaController">
    <div class="container">
        <div id="wiki" class="col-md-6 col-md-offset-3 myvertical mya">
            <h1 class="text-center">Wikipedia Viewer</h1>
            <div class="row">
                <div class="col-md-4 col-md-offset-4 col-xs-8 col-xs-offset-2">
                    <form>
                        <input type="text" class="textinput" placeholder="Search Wikipedia" ng-model="searchTerm" ng-change="searchWikipedia()">
                    </form>
                </div>
            </div>
            <p class="text-center"><a href="http://en.wikipedia.org/wiki/Special:Random" target="_blank" class="randomSearch">Or click here for a random article</a></p>
            <a href="https://en.wikipedia.org/?curid={{ result.pageid }}" target="_blank" ng-repeat="result in results">
                <div class="callout large">
                    <h2>{{ result.title }}</h2>
                    <p>{{ result.extract }}</p>
                </div>
            </a>
        </div>
    </div>
</body>

</html>
```

**main.css**

```


body,
h1,
h2 {
    font-family: 'Raleway', sans-serif;
}

.mya a{
    text-decoration: none;

}

.myvertical {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}

.callout:hover {
    background-color: #EBEBEB;
}

.callout{
    color: #000;
    border: 1px solid #E3E3E3;
    padding: 30px 60px;
    margin-bottom: 20px;
}

.randomSearch{
    font-size: 1.5rem;
}

h1 {
    font-size: 6rem;
    font-weight: 500;
    font-family: 'Raleway', sans-serif;
}

h2 {
    text-transform: uppercase;
}

.textinput{
    margin-top: 10px;
    margin-bottom: 20px;
    height: 35px;
    text-decoration: none;
}
```

**app.js**

```
var app = angular.module('WikipediaApplication', []);

app.controller('WikipediaController', ['$scope', '$http', function($scope, $http) {
    $scope.results = [];

    $scope.searchWikipedia = function() {
        var api = 'http://en.wikipedia.org/w/api.php?action=query&generator=search&prop=extracts&exintro&exsentences=1&exlimit=max&explaintext&gsrsearch=';
        var callback = '&format=json&callback=JSON_CALLBACK';
        $http.jsonp(api + $scope.searchTerm + callback
        ).success(function(data) {
            document.getElementById('wiki').classList.remove('myvertical');
            $scope.results = data.query.pages;
        }).error(function(data) {
            console.log(data);
        });

        if ($scope.searchTerm === '') $scope.results = [];
    }
}]);
```

**Souce:**[WIKISearch](https://github.com/whuhan2013/freeCodeCampProject/tree/master/wiki)      
**liveDemo:**[WIKI](https://whuhan2013.github.io/web/wiki/index.html)

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p4.png)











