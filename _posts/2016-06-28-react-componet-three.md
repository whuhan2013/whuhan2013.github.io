---
layout: post
title: React Native常用组件(三)
date: 2016-6-28
categories: blog
tags: [React Native]
description: React Native实例
---


**本文主要包括以下内容** 

1. ToolbarAndroid控件 
2. Switch开关控件    
3. Picker选择控件


### ToolbarAndroid控件


该ToolBarAndroid组件进行封装了Android平台中的ToolBar组件(只适用于Android平台)。一个ToolBar组件可以显示一个Logo图标以及一些导航图片(例如:菜单功能按钮)，一个标题以及副标题还有一系列功能的列表。标题和副标题是上下位置。所以logo图标和导航图标显示在左边，标题和副标题显示在中间，功能列表显示在右边。


**实例代码**

```
'use strict';
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
} from 'react-native';
var ToolbarAndroid = require('ToolbarAndroid');
class ToolBarAndroidDemo extends Component {
  render() {
    return (
       <ToolbarAndroid
            actions={toolbarActions}
            navIcon={require('./ic_menu_black_24dp.png')}
            style={styles.toolbar}
            subtitle="副标题"
            title="主标题"></ToolbarAndroid>
    );
  }
}
var toolbarActions = [
  {title: 'Create', icon: require('./ic_create_black_48dp.png'), show: 'always'},
  {title: 'Filter'},
  {title: 'Settings', icon: require('./ic_settings_black_48dp.png'), show: 'always'},
];
const styles = StyleSheet.create({
  toolbar: {
    backgroundColor: '#e9eaed',
    height: 56,
  },
});
AppRegistry.registerComponent('ToolBarAndroidDemo', () => ToolBarAndroidDemo);
```

**效果如下** 

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/01/130.jpg)


### Switch控件


Switch属性方法介绍(这边只介绍平台通用属性以及只适合Android平台上面的属性方法)

- View相关属性样式全部继承(例如:宽和高,背景颜色,边距等相关属性样式)
- disabled bool 如果该值为true,用户就无法点击switch开关控件，默认为false
- onValueChange function 方法，当该组件的状态值发生变化的时候回调方法
- value bool 该开关的值，如果该值为true的时候，开关呈打开状态，默认为false   



```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Image,
  Text,
  TextInput,
   ScrollView,
    TouchableOpacity,
    ToolbarAndroid,
  DrawerLayoutAndroid,
   ProgressBarAndroid,
  Switch,
  View
} from 'react-native';
class MyProject extends Component {

 render() {
  return (
        <SwitchDemo></SwitchDemo>
    );
}


}


var SwitchDemo = React.createClass({
  getInitialState() {
    return {
      trueSwitchIsOn: true,
      falseSwitchIsOn: false,
    };
  },
  render() {
    return (
      <View style={styles.container}>
        <Text>
           Swtich实例
        </Text>
        <Switch
          onValueChange={(value) => this.setState({falseSwitchIsOn: value})}
          style={{marginBottom:10,marginTop:10}}
          value={this.state.falseSwitchIsOn} />
        <Switch
          onValueChange={(value) => this.setState({trueSwitchIsOn: value})}
          value={this.state.trueSwitchIsOn} />
      </View>
    );
  }
});

const styles = StyleSheet.create({
    container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  toolbar: {
    backgroundColor: '#48D1CC',
    height: 56,
  },
});

AppRegistry.registerComponent('MyProject', () => MyProject);
```

**效果如下** 

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/01/133.jpg)


### Picker选择控件 

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,

  StyleSheet,
  Text,
  View,
  Picker,
} from 'react-native';
class MyProject extends Component {

 render() {
  return (
        <PickerDemo></PickerDemo>
    );
}


}


var PickerDemo = React.createClass({
   getInitialState: function() {
    return {
      language: '',
    };
  },
  render() {
    return (
      <View style={styles.container}>
        <Text >
          Picker选择器实例
        </Text>
        <Picker 
          style={{width:200}}
          selectedValue={this.state.language}
          onValueChange={(value) => this.setState({language: value})}>
          <Picker.Item label="Java" value="java" />
          <Picker.Item label="JavaScript" value="javaScript" />
        </Picker>
        <Text>当前选择的是:{this.state.language}</Text>
      </View>
    );
  }
});

const styles = StyleSheet.create({
    container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
});

AppRegistry.registerComponent('MyProject', () => MyProject);
```


**效果如下**

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/01/3.2.jpg)