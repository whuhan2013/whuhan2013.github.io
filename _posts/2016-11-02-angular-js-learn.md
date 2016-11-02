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
- 依赖管理工具bower
- 轻量级server http-server
- 单元测试runner-karma
- 单元测试工具－jasmine

#### MVC

controller的重复利用，将共同部分抽取出来 

```
<!doctype html>
<html ng-app>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <div ng-controller="CommonController">
            <div ng-controller="Controller1">
                <p>{{greeting.text}},Angular</p>
                <button ng-click="test1()">test1</button>
            </div>
            <div ng-controller="Controller2">
                <p>{{greeting.text}},Angular</p>
                <button ng-click="test2()">test2</button>
                <button ng-click="commonFn()">通用</button>
            </div>
        </div>
    </body>
    <script src="js/angular-1.3.0.js"></script>
    <script src="MVC3.js"></script>
</html>


function CommonController($scope){
  $scope.commonFn=function(){
      alert("这里是通用功能！");
    };
}

function Controller1($scope) {
    $scope.greeting = {
        text: 'Hello1'
    };
    $scope.test1=function(){
      alert("test1");
    };
}

function Controller2($scope) {
    $scope.greeting = {
        text: 'Hello2'
    };
    $scope.test2=function(){
      alert("test2");
    }
}
```

controller使用过程中的注意点：        
1、不要试图去复用controller,一个控制器一般只负责一小块视图         
2、不要在controller中操作dom，这不是控制器的职责           
3、不要在controller里面做数据格式化，ng有很好的表单控件         
4、不要在controller中做数据过滤操作，ng有$filter服务                
5、一般来说，controller是不会互相调用的，控制器之间的交互会通过事件进行      

**angulas的mvc借助于$scope实现** 

scope的作用域      

```
function GreetCtrl($scope, $rootScope) {
  $scope.name = 'World';
  $rootScope.department = 'Angular';
}

function ListCtrl($scope) {
  $scope.names = ['Igor', 'Misko', 'Vojta'];
}
```

rootScope是全局的      

1、$scope是一个POJO（plain old javascript object）        
2、$scope提供了一些工具方法$watch()/$apply()          
3、$scope是表达式的执行环境             
4、$scope是一个树型结构，与dom标签平行        
5、子$scope对象会继承父$scope上的属性和方法               
6、每一个angular应用只有一个根$scope对象（一般位于ng-app上）         
7、$scope可以传播事件，类似dom事件，可以向上也可以向下       
8、$scope不仅是MVC的基础，也是后面实现双向数据绑定的基础       
9、可以用angular.element($0).scope()进行调试       



#### 路由、模块、依赖注入















