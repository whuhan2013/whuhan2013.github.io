---
layout: post
title: React Native API模块
date: 2016-6-29
categories: blog
tags: [React Native]
description: React Native实例
---


**本文主要包括以下内容** 

1. ToastAndroid模块
2. Alert模块   

### ToastAndroid模块

该ToastAndroid模块是把Android平台原生模块Toast封装成一个JS模块使用，来进行弹出一个toast消息。该模块有一个'show'方法会传入下面两个参数:①:message  string 字符串格式，设置要进行toast显示的文本    ②:duration  int格式 toast消息弹出显示的时长。有两个可选值ToastAndroid.SHORT或者ToastAndroid.LONG


**example** 


```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 */
'use strict';
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  ToastAndroid,
  TouchableHighlight,
} from 'react-native';
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
class ToastAndroidDemo extends Component {
  render() {
    return (
      <View>
        <Text style={styles.welcome}>
          点击弹出短时间的toast
        </Text>
        <CustomButton text="点击弹出短时间toast"
        onPress={()=>ToastAndroid.show('点击我好疼,短时间的~',ToastAndroid.SHORT)}/>
        <Text style={styles.welcome}>
          点击弹出长时间的toast
        </Text>
        <CustomButton text="点击弹出长时间toast"
        onPress={()=>ToastAndroid.show('点击我好疼,长时间的~',ToastAndroid.LONG)}/>
      </View>
    );
  }
}
const styles = StyleSheet.create({
  button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  }
});
 
AppRegistry.registerComponent('ToastAndroidDemo', () => ToastAndroidDemo);
```

**effect**

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/02/one4.gif)


### Alert模块

Alert模块是创建弹出一个弹框，显示一个标题以及相关信息内容。该弹出框可以提供一系列的可选按钮，点击任何一个按钮都会调用onPress方法并且关闭弹框。默认情况下，该只会显示一个'确定'按钮。


iOS平台可以指定多个数量的button，但是在Android平台上面最多只能指定三个按钮。Android平台的弹出框的按钮有'中间态','取消','确认'三种状态

1.如果你只有指定了一个按钮，那么该为'positive' (例如:确定)

2.如果你指定了两个按钮，那么该会'negative','positive' (例如:确定，取消)

3.如果你指定了三个按钮，那么该会'neutral','negative','positive'(例如:稍后再说,'确定','取消')


**example** 


```
/**
 * React Native Alert模块具体使用实例
 * https://github.com/facebook/react-native
 */
'use strict';
import React, {
  AppRegistry,
  Component,
  StyleSheet,
  Text,
  View,
  Alert,
  ToastAndroid,
  TouchableHighlight,
} from 'react-native';
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
class AlertDemo extends Component {
  render() {
    return (
      <View>
        <Text style={{height:30,color:'black',margin:8}}>
          弹出框实例
        </Text>
        <CustomButton text='点击弹出默认Alert'
          onPress={()=>Alert.alert('温馨提醒','确定退出吗?')}
          />
        <CustomButton text='点击弹出两个按钮的Alert'
          onPress={()=>Alert.alert('温馨提醒','确定退出吗?',[
            {text:'取消',onPress:()=>ToastAndroid.show('你点击了取消~',ToastAndroid.SHORT)},
            {text:'确定',onPress:()=>ToastAndroid.show('你点击了确定~',ToastAndroid.SHORT)}
            ])}
          />
        <CustomButton text='点击弹出三个按钮的Alert'
          onPress={()=>Alert.alert('温馨提醒','确定退出吗?',[
            {text:'One',onPress:()=>ToastAndroid.show('你点击了One~',ToastAndroid.SHORT)},
            {text:'Two',onPress:()=>ToastAndroid.show('你点击了Two~',ToastAndroid.SHORT)},
            {text:'Three',onPress:()=>ToastAndroid.show('你点击了Three~',ToastAndroid.SHORT)}
            ])}
          />
      </View>
    );
  }
}
const styles = StyleSheet.create({
  button: {
    margin:5,
    backgroundColor: 'white',
    padding: 15,
    borderBottomWidth: StyleSheet.hairlineWidth,
    borderBottomColor: '#cdcdcd',
  }
});
 
AppRegistry.registerComponent('AlertDemo', () => AlertDemo);
```

**effect**

![](http://lookcode-wordpress.stor.sinaapp.com/uploads/2016/02/one5.gif)