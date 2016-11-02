---
layout: post
title: AngularJS学习
date: 2016-11-2
categories: blog
tags: [javascript]
description: AngularJS学习
---

#### angular4大特性    

1：mvc      
2:模块化    
3:指令系统     
4：双向数据绑定

**AngularJS MVC实现**     

view

```
<!DOCTYPE html>
<html ng-app="angularApp">

<head>
    <title>Angular</title>
    <!-- Angular JS -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.min.js"></script>
    <!-- Custom JS -->
    <script type="text/javascript" src="js/app.js"></script>
</head>

<body>
    <div ng-controller="HelloAngular">
        <p>{{greeting.text}},Angular</p>
    </div>
</body>

</html>
```

controll       
angular从1.3几开始就不支持全局controller,需要创建module


```
var myModule = angular.module("angularApp", []);

myModule.controller("HelloAngular", function($scope){

    $scope.greeting = {
            text: 'Hello1rwesfs'
        };

});
```

#### 指令系统实现

view

```
<!doctype html>
<html ng-app="MyModule">
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <hello>123</hello>
  </body>
  <script src="js/angular-1.3.0.js"></script>
  <script src="HelloAngular_Directive.js"></script>
</html>
```

controller

```
var myModule = angular.module("MyModule", []);
myModule.directive("hello", function() {
    return {
        restrict: 'E',
        template: '<div>Hi everyone!</div>',
        replace: true
    }
});
```


#### 双向数据绑定

当model发生改变时，界面会自动发生变化

```
<!doctype html>
<html ng-app>

<head>
    <meta charset="utf-8">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.min.js"></script>
</head>

<body>
    <div>
        <input ng-model="greeting.text" />
        <p>{{greeting.text}},AngularJS</p>
    </div>
</body>

</html>
```


#### 前端开发工具        

- 代码编辑工具sublime      
- 断点调试工具chrome插件Batarang
- 版本管理工具git
- 开发与调试开具nodejs
- 代码合并与混淆工具grunt      
- 

















