---
layout: post
title: React Native实例
date: 2016-6-24
categories: blog
tags: [React Native]
description: React Native实例
---


**效果如下** 

![这里写图片描述](http://img.blog.csdn.net/20160624155046443)


**代码如下** 

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
    PixelRatio,
  View
} from 'react-native';

class Abc extends Component {
  render() {
    return (
     <View style={styles.flex}>
      <View style={styles.container}>
        <View style={[styles.item,styles.center]}>

          <Text style={styles.font}>酒店</Text>

        </View>

        <View style={[styles.item,styles.lineLeftRight]}>

          <View style={[styles.center,styles.flex,styles.lineCenter]}>
            <Text style={styles.font}>海外酒店</Text>
          </View>

          <View style={[styles.center,styles.flex]}>
            <Text style={styles.font}>特惠酒店</Text>

          </View>



        </View>



        <View style={styles.item}>

          <View style={[styles.center,styles.flex,styles.lineCenter]}>
            <Text style={styles.font}>团购</Text>
          </View>

          <View style={[styles.center,styles.flex]}>
            <Text style={styles.font}>客栈，公寓</Text>

          </View>




        </View>
      </View>

        </View>
    );
  }
}

const styles = StyleSheet.create({
 container: {
    marginTop:200,
    marginLeft:5,
    marginRight:5,
    height:84,
    flexDirection:'row',
    borderRadius:5,
    padding:2,
    backgroundColor:'#FF0067',
  },
  item: {
    flex: 1,
    height:80,

  },
  center:{
    justifyContent:'center',
    alignItems:'center',
  },
  flex:{
    flex:1,
  },
  font:{
    color:'#fff',
    fontSize:16,
    fontWeight:'bold',
  },
  lineLeftRight:{
    borderLeftWidth:1/PixelRatio.get(),
    borderRightWidth:1/PixelRatio.get(),
    borderColor:'#fff',
  },
  lineCenter:{
    borderBottomWidth:1/PixelRatio.get(),
    borderColor:'#fff',
  },
});

AppRegistry.registerComponent('Abc', () => Abc);
```


### 网易新闻实例 

**head.js** 

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
    PixelRatio,
  View
} from 'react-native';

class Header extends Component {
  render() {
    return (
       <View style={styles.flex}>
                <Text style={styles.font}>

                    <Text style={styles.font_1}>网易</Text>
                    <Text style={styles.font_2}>新闻</Text>
                    <Text>有态度"</Text>
                </Text>

        </View>
    );
  }
}

const styles = StyleSheet.create({
  flex:{
        marginTop:25,
        height:50,
        borderBottomWidth:3/PixelRatio.get(),
        borderBottomColor:'#EF2D36',
        alignItems:'center',
    },
    font:{
        fontSize:25,
        fontWeight:'bold',
        textAlign:'center',
    },
    font_1:{
        color:'#CD1D1C',
    },
    font_2:{
        color:'#FFF',
        backgroundColor:'#CD1D1C',
    },
});

module.exports=Header;
```


将head.js导入到主JS中的代码为const Header=require('./head');

**主JS详细代码**

实现了列表，并且有点击效果 

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
    PixelRatio,
  View
} from 'react-native';

const Header=require('./head');

class Abc extends Component {
  render() {
    return (
     <View style={styles.flex}>
            <Header></Header>
            <List title='一线城市楼市退烧 有房源一夜跌价160万'></List>
            <List title='上海市民称墓地太贵买不起 买房存骨灰'></List>
            <List title='朝鲜再发视频:摧毁青瓦台 一切化作灰烬'></List>
            <List title='生活大爆炸人物原型都好牛逼'></List>

             <ImportantNews
                news={[
                '解放军报报社大楼正在拆除 标识已被卸下(图)',
                '韩国停签东三省52家旅行社 或为阻止朝旅游创汇',
                '南京大学生发起亲吻陌生人活动 有女生献初吻-南京大学生发起亲吻陌生人活动 有女生献初吻-南京大学生发起亲吻陌生人活动 有女生献初吻',
                '防总部署长江防汛:以防御98年量级大洪水为目标'
                ]}>
            </ImportantNews>
      </View>

    );
  }
}


class List extends Component{
    render(){
        return (
            <View style={styles.list_item}>
               <Text style={styles.list_item_font}>{this.props.title}</Text>
            </View>
        );
    }
}


class ImportantNews extends Component{

    show(title){

        alert(title);

    }


    render(){
        var news=[];
        for(var i in this.props.news){
            var text=(
                <Text
                    onPress={this.show.bind(this,this.props.news[i])}
                    numberOfLines={2}
                    style={styles.news_item}
                    key={i}

                    >{this.props.news[i]}</Text>
            );
            news.push(text);
        }
        return (
            <View style={styles.flex}>
                <Text style={styles.news_title}>今日要闻</Text>
                {news}

            </View>

        );
    }
}

const styles = StyleSheet.create({
  flex:{
    flex:1,
  },

    list_item:{
        height:40,
        marginLeft:10,
        marginRight:10,
        borderBottomWidth:1,
        borderBottomColor:'#ddd',
        justifyContent:'center',
    },

    list_item_font:{
        fontSize:16,
    },

    news_item:{
        marginLeft:10,
        marginRight:10,
        fontSize:15,
        lineHeight:40,
    },

    news_title:{
        fontSize:20,
        fontWeight:'bold',
        color:'#CD1D1C',
        marginLeft:10,
        marginTop:15,

    },
});

AppRegistry.registerComponent('Abc', () => Abc);
```


**效果如下** 


![这里写图片描述](http://img.blog.csdn.net/20160624182446906)

![这里写图片描述](http://img.blog.csdn.net/20160624182458596)