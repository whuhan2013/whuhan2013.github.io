---
layout: post
title: React Native常用组件(四)
date: 2016-6-29
categories: blog
tags: [React Native]
description: React Native实例
---


**本文主要包括以下内容** 

1. WebView组件  
2. 


### WebView组件 

WebView组件最基本的使用方法，直接加载一个网页

```
'use strict';
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  WebView,
} from 'react-native';
var DEFAULT_URL = 'http://www.lcode.org';
 
var WebViewDemo = React.createClass({
  render: function() {
    return (
      <View style={{flex:1}}>
        <Text style={{height:40}}>简单的网页显示</Text>
        <WebView style={styles.webview_style} 
          url={DEFAULT_URL}
          startInLoadingState={true}
          domStorageEnabled={true}
          javaScriptEnabled={true}
          >
        </WebView>
      </View>
    );
  },
});
var styles = StyleSheet.create({
    webview_style:{  
       backgroundColor:'#00ff00',   
    }
});
 
AppRegistry.registerComponent('WebViewDemo', () => WebViewDemo);
```


**effect**

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/02/one1.gif)

**WebView加载本地的HTML静态字符串**

```
'use strict';
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  WebView,
} from 'react-native';
var DEFAULT_URL = 'http://www.lcode.org';
const HTML = `
<!DOCTYPE html>\n
<html>
  <head>
    <title>HTML字符串</title>
    <meta http-equiv="content-type" content="text/html; charset=utf-8">
    <meta name="viewport" content="width=320, user-scalable=no">
    <style type="text/css">
      body {
        margin: 0;
        padding: 0;
        font: 62.5% arial, sans-serif;
        background: #ccc;
      }
      h1 {
        padding: 45px;
        margin: 0;
        text-align: center;
        color: #33f;
      }
    </style>
  </head>
  <body>
    <h1>加载静态的HTML文本信息</h1>
  </body>
</html>
`;
var WebViewDemo = React.createClass({
  render: function() {
    return (
      <View style={{flex:1}}>
        <WebView style={styles.webview_style} 
          html={HTML}
          startInLoadingState={true}
          domStorageEnabled={true}
          javaScriptEnabled={true}
          >
        </WebView>
      </View>
    );
  },
});
var styles = StyleSheet.create({
    webview_style:{  
       backgroundColor:'#00ff00',   
    }
});
 
AppRegistry.registerComponent('WebViewDemo', () => WebViewDemo);
```

**effect** 

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/02/two.jpg)