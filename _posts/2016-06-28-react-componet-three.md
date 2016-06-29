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
4. ViewPagerAndroid组件
5. Touchable组件
6. RefreshControl组件


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


### ViewPagerAndroid组件 

ViewPagerAndroid中的所有子View必须为<View>控件，不能为复合型的组件。你可以为每一个子视图添加列如:padding或则backgroundColor之类的属性。 



```
'use strict';
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  ViewPagerAndroid,
} from 'react-native';
 
class ViewPagerDemo extends Component {
  render() {
    return (
      <View >
        <Text style={styles.welcome}>
            ViewPagerAndroid实例
        </Text>
        <ViewPagerAndroid style={styles.pageStyle} initialPage={0}>
             <View style={{backgroundColor:"red"}}>
                   <Text>First Page!</Text>
             </View>
             <View style={{backgroundColor:"yellow"}}>
                   <Text>Second Page!</Text>
             </View>
        </ViewPagerAndroid>
      </View>
    );
  }
}
const styles = StyleSheet.create({
   pageStyle: {
    alignItems: 'center',
    padding: 20,
    height:200,
  }
});
AppRegistry.registerComponent('ViewPagerDemo', () => ViewPagerDemo);
```

**效果如下** 

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/02/1.jpg)

### Touchable组件 


今天我们一起来看一下Touchable*系列组件的使用详解，该系列组件包括四种分别为:TouchableHighlight,TouchableNativeFeedback,TouchableOpacity,TouchableWithoutFeedback。其中最后一个控件是触摸点击不带反馈效果的，另外三个都是有反馈效果。可以这样理解前面三个都是继承自TouchableWithoutFeedback扩展而来。


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
  Image,
  TouchableHighlight,
  ViewPagerAndroid,
   TouchableOpacity,
   TouchableWithoutFeedback,
} from 'react-native';
class MyProject extends Component {
render() {
    return (
       <View style={styles.flex}>

          <TouchableHighlight onPress={this.show.bind(this,'欢迎学习React Native技术')}
              underlayColor='#E1F6FF'
              >
          <Text style={styles.item}>欢迎学习React Native技术-TouchableHighlight</Text>

          </TouchableHighlight>

          <TouchableOpacity onPress={this.show.bind(this,'作者东方耀Q：3096239789')}>
          <Text style={styles.item}>作者东方耀Q：3096239789-TouchableOpacity</Text>

          </TouchableOpacity>


          <TouchableWithoutFeedback onPress={this.show.bind(this,'作者东方耀Q：3096239789')}>
            <Text style={styles.item}>作者东方耀Q：3096239789-TouchableWithoutFeedback</Text>


          </TouchableWithoutFeedback>

        </View>
    );
  }

   show(txt){
    alert(txt);
  }


}






const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
   
  },
  heiligt:
  {
    borderRadius: 8,
    padding: 6,
    marginTop:5,
  

  },
   flex:{
    flex:1,
    marginTop:25,
  },

 item:{
   fontSize:18,
   color:'#434343',
   marginLeft:5,
   marginTop:10,
 },
});

AppRegistry.registerComponent('MyProject', () => MyProject);
```


**注意**

必须要有onPress方法，不然的话不会产生点击效果，也就不会产生变化了。


### RefreshControl组件

This component is used inside a ScrollView or ListView to add pull to refresh functionality. When the ScrollView is at scrollY: 0, swiping down triggers an onRefresh event.


**Examples** 

```
'use strict';
 
const React = require('react-native');
const {
  AppRegistry,
  ScrollView,
  StyleSheet,
  RefreshControl,
  Text,
  View,
} = React;
 
const styles = StyleSheet.create({
  row: {
    borderColor: 'red',
    borderWidth: 5,
    padding: 20,
    backgroundColor: '#3a5795',
    margin: 5,
  },
  text: {
    alignSelf: 'center',
    color: '#fff',
  },
  scrollview: {
    flex: 1,
  },
});
 
const Row = React.createClass({
  _onClick: function() {
    this.props.onClick(this.props.data);
  },
  render: function() {
    return (
        <View style={styles.row}>
          <Text style={styles.text}>
            {this.props.data.text}
          </Text>
        </View>
    );
  },
});
 
const RefreshControlDemo = React.createClass({
  getInitialState() {
    return {
      isRefreshing: false,
      loaded: 0,
      rowData: Array.from(new Array(20)).map(
        (val, i) => ({text: '初始行 ' + i})),
    };
  },
  render() {
    const rows = this.state.rowData.map((row, ii) => {
      return <Row key={ii} data={row}/>;
    });
    return (
      <ScrollView
        style={styles.scrollview}
        refreshControl={
          <RefreshControl
            refreshing={this.state.isRefreshing}
            onRefresh={this._onRefresh}
            colors={['#ff0000', '#00ff00', '#0000ff','#3ad564']}
            progressBackgroundColor="#ffffff"
          />
        }>
        {rows}
      </ScrollView>
    );
  },
  _onRefresh() {
    this.setState({isRefreshing: true});
    setTimeout(() => {
      // 准备下拉刷新的5条数据
      const rowData = Array.from(new Array(5))
      .map((val, i) => ({
        text: '刷新行 ' + (+this.state.loaded + i)
      }))
      .concat(this.state.rowData);
 
      this.setState({
        loaded: this.state.loaded + 5,
        isRefreshing: false,
        rowData: rowData,
      });
    }, 5000);
  },
});
 
AppRegistry.registerComponent('RefreshControlDemo', () => RefreshControlDemo);
```


**References**

[函数的扩展 - ECMAScript 6入门](http://es6.ruanyifeng.com/#docs/function)           
[Array.prototype.map() - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)                 

**effect**

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/02/demo1.gif)