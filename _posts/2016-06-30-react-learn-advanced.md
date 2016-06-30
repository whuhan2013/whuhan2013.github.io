---
layout: post
title: React Native进阶
date: 2016-6-30
categories: blog
tags: [React Native]
description: React Native进阶
---


**本文主要包括以下内容** 

1. 原生模块封装  
2. 回调方法   

test

### 原生模块封装

有时候我们的应用需要进行访问原生平台系统的API接口，但是React Native可能还没有封装相应功能组件。还有可能我们需要去复用一些原生java代码而不是让JavaScript重新去实现一遍。或者我们可能需要些一些更加高级的功能代码，所线程相关的。例如:图片处理,数据库以及一些高级功能扩展之类的。

React Native平台的开发其实本身也是可以让你写纯原生代码并且还可以让你访问原生平台的功能。这是一个比较高级的功能不过官方还是不推荐你在平时开发中使用这样的开发形式。但是如果你具备这样的开发能力，也是还是不错的。特别当在React Native暂时未提供部分原生功能或者模块，那么你可以使用这样的方法进行封装扩展。今天我们就来看一下原生组件的封装扩展方法。


**example**


现在我们来拿Android原生Toast模块进行举例子。进行封装之后，通过JavaScript来进行弹出toast消息。

首先我们需要创建一个原生模块类，该原生模块类是继承ReactContextBaseJavaModule类的Java代码，该用来实现JavaScript所需的一些功能。我们的目标是使用JavaScript代码通过调用ToastAndroid.show('Awesome',ToastAndroid.SHORT)方法进行再屏幕上面显示一个toast消息提醒。(注意:最终我这边演示的方法调用方法可能和文档中有所区别,因为没有加入module化)


```
package com.modules.custom;

import android.widget.Toast;

import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.uimanager.IllegalViewOperationException;

import java.util.HashMap;
import java.util.Map;

/**
 * 当前类注释:测试原生Toast模块
 * 项目名：android
 * 包名：com.modules.custom
 * 作者：江清清 on 16/3/31 10:18
 * 邮箱：jiangqqlmj@163.com
 * QQ： 781931404
 * 来源：<a href="http://www.lcode.org">江清清的技术专栏</>
 */
public class ToastCustomModule extends ReactContextBaseJavaModule {

    private static final String DURATION_SHORT="SHORT";
    private static final String DURATION_LONG="LONG";
    public ToastCustomModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    @Override
    public String getName() {
        return "ToastCustomAndroid";
    }
    @Override
    public Map<String, Object> getConstants() {
        final Map<String, Object> constants = new HashMap<>();
        constants.put(DURATION_SHORT, Toast.LENGTH_SHORT);
        constants.put(DURATION_LONG, Toast.LENGTH_LONG);
        return constants;
    }

    /**
     * 该方法用于给JavaScript进行调用
     * @param message
     * @param duration
     */
    @ReactMethod
    public void show(String message, int duration) {
        Toast.makeText(getReactApplicationContext(), message, duration).show();
    }

    /**
     * 这边只是演示相关回调方法的使用,所以这边的使用方法是非常简单的
     * @param errorCallback       数据错误回调函数
     * @param successCallback     数据成功回调函数
     */
    @ReactMethod
    public void measureLayout(Callback errorCallback,
                              Callback successCallback){
        try {
            successCallback.invoke(100, 100, 200, 200);
        } catch (IllegalViewOperationException e) {
            errorCallback.invoke(e.getMessage());
        }
    }
}
```


**模块注册** 

```
package com.modules.custom;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.JavaScriptModule;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

/**
 * 当前类注释:测试原生Toast模块
 * 项目名：android
 * 包名：com.modules.custom
 * 作者：江清清 on 16/3/31 10:18
 * 邮箱：jiangqqlmj@163.com
 * QQ： 781931404
 * 来源：<a href="http://www.lcode.org">江清清的技术专栏</>
 */
public class AnToastReactPackage implements ReactPackage {
    @Override
    public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();
        modules.add(new ToastCustomModule(reactContext));
        return modules;
    }

    @Override
    public List<Class<? extends JavaScriptModule>> createJSModules() {
        return Collections.emptyList();
    }

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }
}
```


接下来，上面自定义的packager需要在MainActivity.java中的getPackagers方法中进行添加(具体路径:android/app/src/main/java/com/your-app-name/MainActivity.java)。具体参考代码如下:

```
/**
     * A list of packages used by the app. If the app uses additional views
     * or modules besides the default ones, add more packages here.
     */
    @Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
                new AnToastReactPackage()
        );
    }
```


不过为了在JavaScript实现当前封装的模块可以使用方便，通常会把该封装成JavaScript模块。这样可以省下每次访问组件都要加入NativeModules的步骤。具体方式如下:


```
'use strict';
 
/**
 * This exposes the native ToastAndroid module as a JS module. This has a function 'show'
 * which takes the following parameters:
 *
 * 1. String message: A string with the text to toast
 * 2. int duration: The duration of the toast. May be ToastAndroid.SHORT or ToastAndroid.LONG
 */
var { NativeModules } = require('react-native');
module.exports = NativeModules.ToastCustomAndroid;
```


然后在别的JavaScript文件代码中就可以如下进行访问了:

```
var ToastCustomAndroid= require('./ToastCustomAndroid')
ToastCustomAndroid.show('Awesome', ToastCustomAndroid.SHORT);
```


本文章都是演示效果，所以就没有采用module.exports模式。

那么具体在JavaScript文件中的调用方法实例代码如下:

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
 
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  TouchableHighlight,
} from 'react-native';
var { NativeModules } = require('react-native');
class CustomButton extends React.Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.button}
        underlayColor="#a5a5a5"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>{this.props.text}</Text>
      </TouchableHighlight>
    );
  }
}
class ModulesDemo extends Component {
  render() {
    return (
      <View>
        <Text style={styles.welcome}>
          自定义弹出Toast消息
        </Text>
        <CustomButton
          text="点击弹出Toast消息"
          onPress={()=>NativeModules.ToastCustomAndroid.show("我是ToastCustomAndroid弹出消息",NativeModules.ToastCustomAndroid.SHORT)}
        />
      </View>
    );
  }
}
const styles = StyleSheet.create({
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
   button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  },
});
 
AppRegistry.registerComponent('ModulesDemo', () => ModulesDemo);
```

**effect** 

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/03/toast.gif)



### 回调方法


原生模块有一种特殊的参数那就是回调函数，在绝大多数情况下该用来提供一个回调方法进行传值给JavaScript。


**example** 


```
public class ToastCustomModule extends ReactContextBaseJavaModule {
 
    private static final String DURATION_SHORT="SHORT";
    private static final String DURATION_LONG="LONG";
    public ToastCustomModule(ReactApplicationContext reactContext) {
        super(reactContext);
    }
    @Override
    public String getName() {
        return "ToastCustomAndroid";
    }
    @Override
    public Map<String, Object> getConstants() {
        final Map<String, Object> constants = new HashMap<>();
        constants.put(DURATION_SHORT, Toast.LENGTH_SHORT);
        constants.put(DURATION_LONG, Toast.LENGTH_LONG);
        return constants;
    }
 
    /**
     * 该方法用于给JavaScript进行调用
     * @param message
     * @param duration
     */
    @ReactMethod
    public void show(String message, int duration) {
        Toast.makeText(getReactApplicationContext(), message, duration).show();
    }
 
    /**
     * 这边只是演示相关回调方法的使用,所以这边的使用方法是非常简单的
     * @param errorCallback       数据错误回调函数
     * @param successCallback     数据成功回调函数
     */
    @ReactMethod
    public void measureLayout(Callback errorCallback,
                              Callback successCallback){
        try {
            successCallback.invoke(100, 100, 200, 200);
        } catch (IllegalViewOperationException e) {
            errorCallback.invoke(e.getMessage());
        }
    }
}
```


具体前端JavaScript文件中的调用方式如下:


```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
 
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  TouchableHighlight,
  ToastAndroid,
} from 'react-native';
var { NativeModules } = require('react-native');
class CustomButton extends React.Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.button}
        underlayColor="#a5a5a5"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>{this.props.text}</Text>
      </TouchableHighlight>
    );
  }
}
//onPress={()=>NativeModules.ToastCustomAndroid.show("我是ToastCustomAndroid弹出消息",NativeModules.ToastCustomAndroid.SHORT)}
class ModulesDemo extends Component {
  render() {
    return (
      <View>
        <Text style={styles.welcome}>
          自定义弹出Toast消息
        </Text>
        <CustomButton
          text="点击弹出Toast消息"
          onPress={()=>NativeModules.ToastCustomAndroid.measureLayout((msg) => {
                    console.log(msg);
                  },
                   (x, y, width, height) => {
                    console.log(x + '坐标,' + y + '坐标,' + width + '宽,' + height+'高');
                  })}
        />
      </View>
    );
  }
}
const styles = StyleSheet.create({
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
   button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  },
});
 
AppRegistry.registerComponent('ModulesDemo', () => ModulesDemo);
```


**effect** 

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/04/11.jpg)