---
layout: post
title: React Native实例(二)
date: 2016-6-25
categories: blog
tags: [React Native]
description: React Native实例
---


**本文主要包括以下内容** 

1. listView实例            


### listView实例 

主要讲述了从通过网络获取数据，并且加载到listView中的实例

**ListView的介绍 

Let's now modify this application to render all of this data in a ListView component, rather than just rendering the first movie.

Why is a ListView better than just rendering all of these elements or putting them in a ScrollView? Despite React being fast, rendering a possibly infinite list of elements could be slow. ListView schedules rendering of views so that you only display the ones on screen and those already rendered but off screen are removed from the native view hierarchy.


**代码**

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
   Navigator,
   ScrollView,
   TextInput,
   ListView,
  Text,
  Image,
    PixelRatio,
  View
} from 'react-native';

var REQUEST_URL = 'https://raw.githubusercontent.com/facebook/react-native/master/docs/MoviesExample.json';

var MOCKED_MOVIES_DATA = [
  {title: 'Title', year: '2015', posters: {thumbnail: 'http://i.imgur.com/UePbdph.jpg'}},
];

class Abc extends Component {
    constructor(props) {
    super(props);
     this.state = {
      dataSource: new ListView.DataSource({
        rowHasChanged: (row1, row2) => row1 !== row2,
      }),
      loaded: false,
    };
  }
 fetchData() {
    fetch(REQUEST_URL)
      .then((response) => response.json())
      .then((responseData) => {
        this.setState({
          dataSource: this.state.dataSource.cloneWithRows(responseData.movies),
          loaded: true,
        });
      })
      .done();
  }
  componentDidMount() {
    this.fetchData();
  }
  render() {
      if (!this.state.loaded) {
      return this.renderLoadingView();
    }

   return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderMovie}
        style={styles.listView}
      />
    );
       
    }



renderLoadingView() {
    return (
      <View style={styles.container}>
        <Text>
          Loading movies...
        </Text>
      </View>
    );
  }

  renderMovie(movie1) {
    return (
      <View style={styles.container}>
        <Image
          source={{uri: movie1.posters.thumbnail}}
          style={styles.thumbnail}
        />
        <View style={styles.rightContainer}>
          <Text style={styles.title}>{movie1.title}</Text>
          <Text style={styles.year}>{movie1.year}</Text>
        </View>
      </View>
    );
  }
}






const styles = StyleSheet.create({
 
   container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  thumbnail: {
    width: 53,
    height: 81,
  },
   rightContainer: {
    flex: 1,
  },
  title: {
    fontSize: 20,
    marginBottom: 8,
    textAlign: 'center',
  },
  year: {
    textAlign: 'center',
  },
   listView: {
    paddingTop: 20,
    backgroundColor: '#F5FCFF',
  },
});

AppRegistry.registerComponent('Abc', () => Abc);
```


**效果如下** 

![](https://facebook.github.io/react-native/img/TutorialFinal.png)

