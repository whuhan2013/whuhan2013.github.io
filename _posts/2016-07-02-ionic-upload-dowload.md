---
layout: post
title: ionic项目之上传下载数据
date: 2016-7-2
categories: blog
tags: [Ionic]
description: ionic项目之上传下载数据
---

**首先是上传数据**  

记得在angularjs的controller中注入$http依赖

```
var data = {id: $scope.task_id, 
          groupId: $scope.task_groupid, 
          importance: $scope.priority_level, 
          classname:$scope.classname, 
          title: $scope.task_title, 
          date: $scope.todo_date, 
          detail: $scope.task_detail,
          images:$scope.images_list,
          name:$scope.name,
          record:$scope.recordPath,
          donedate:$scope.done_date}
        
         $http({
          method: 'POST',
          url: 'http://localhost:3000',
          params:{
            'require': data
          }
          
         }).success(function(data,status,headers,config){
          
         }).error(function(data,status,headers,config){
          
         });
```


$http方法中的data参数，服务端没有接收成功         
$http方法中的params参数，服务端接收成功                     
服务端接收方法是 url.parse(req.url, true).query.params 
（params即是加在url?后的参数）     


**然后是下载数据**  


```
    //刷新
    $scope.refresh = function(){
      $http.get("http://localhost:3000/refresh")
      .success(function(data, status, headers, config) {
        // this callback will be called asynchronously
        // when the response is available
        
        alert("success");
        alert("["+data.substring(0,data.lastIndexOf(","))+"]");                                                                                                                                                                                                                                                                  
        TodoListService.deleteAndAddTask(JSON.parse("["+data.substring(0,data.lastIndexOf(","))+"]"));
        TodoListService.findByGroupId($stateParams.groupId).then(function(todolists) {
       
      $scope.todolists = todolists; //对应类别的所有数据list
    });
        
        })
      .error(function(data, status, headers, config) {
        // called asynchronously if an error occurs
        // or server returns response with an error status.
        alert("error "+data);

        });
      
      
//      .success(function(response) {
//
//          console.log(response);
//          alert(response);
//      });
           
    }
```


回调的success方法中返回的data数据，即为服务器response给客户端的数据
注意：返回的json字符串要保证符合json格式（一开始最外层没加中括号就会报错）



**服务端代码** 

服务器端运行时只需要node serve.js即可，就启动了服务器，数据存储在TXT文档中。

```
var http = require('http');
var url = require('url');
var util = require('util');
var fs = require('fs'); 

http.createServer(function(req, res){
  var urldata = req.url+"";
    var require;
  console.log('connet');
  if(urldata.substring(1) == "refresh")
  {
       console.log("refresh");
     require = "";
     fs.readFile('writefile.txt','utf8', function(err, data) { 
      if (err) { 
      console.error(err); 
      } else { 
      console.log(data); 
    
          res.end(data+"");
      } 
       });
       
  }
  else
  {
     console.log("add");
       require = url.parse(req.url, true).query.require+",";
     fs.open('writefile.txt', 'a', function opend(err, fd) { 
     if(err) { 
       console.error(err); 
       return; 
     } 
     var writeBuffer = new Buffer(require),
       bufferPosition = 0,
       bufferLength = writeBuffer.length,
       filePosition = null;
     fs.write(fd,
         writeBuffer,
         bufferPosition,
         bufferLength,
         filePosition,
         function wrote(err,written){
             if(err){throw err;}
         //console.log(err);
       });
       });
  }
  

}).listen(3000);
```


**还有个问题**                   
由于ionic有自己的服务用于启动项目，它启动了http://localhost:8100 的服务，而ionic项目访问服务器需要另外的端口
所以在电脑浏览器调试时会报错 

```
 XMLHttpRequest cannot load http://localhost:3000/refresh. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8100' is therefore not allowed access.
 ```

需要让浏览器忽略跨域问题就行了，可以干脆关闭所有安全策略，当然浏览器会提示你稳定性和安全性降低
如果不想每次启动都用命令行，也可以把启动参数添加到浏览器快捷方式里：右键点击Chrome图标，在“快捷方法”标签页的“目标”一栏添加启动参数到末尾即可（先留一个空格）


**Codes**

[http://download.csdn.net/detail/superjunjin/8487079](http://download.csdn.net/detail/superjunjin/8487079)


**references**

[ionic项目之上传下载数据 - superjunjin的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/superjunjin/article/details/44158567)

[Cordova 3.x 实例开发 -- 基于Ionic的Todo应用 - Fish Where The Fish Are - ITeye技术网站](http://rensanning.iteye.com/blog/2072034)


**effect**

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic77.jpg)


![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic8.jpg)
