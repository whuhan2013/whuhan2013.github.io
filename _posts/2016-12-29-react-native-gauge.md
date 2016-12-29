---
layout: post
title: React Native仪表盘实现
date: 2016-12-29
categories: blog
tags: [React Native]
description: 仪表盘 
---

主要包括指针与刻度的画图，利用了react native art库

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
  PanResponder
} from 'react-native';

import { AnimatedGaugeProgress, GaugeProgress } from 'react-native-simple-gauge';
const MAX_POINTS = 100;
class example extends Component {

   constructor() {
    super();
    var mydata = new Array();
    var smalldata = new Array();
    for(var i=0;i<=100;i=i+10) { 
      mydata[i]=i*2.7-135;
      if(i==0){
        mydata[i]=mydata[i]+1
      }
      if(i==100){
        mydata[i]= mydata[i]-1
      }
    }

    for (var i=2;i<100;i=i+2){
       if(i%10==0){

       }else{
         smalldata[i]=i*2.7-135;
       }
    }

    this.state = {
      isMoving: false,
      pointsDelta: 0,
      points: 0,
      data: 74,
      datas : mydata,
      sdatas : smalldata
    };
  }




   hourHandStyles() {
    return {
      width: 0,
      height: 0,
      position: 'absolute',
      backgroundColor: this.props.hourHandColor,
      top: this.props.clockSize/2+20,
      left: this.props.clockSize/2+20,
      marginVertical: -this.props.hourHandWidth,
      marginLeft: -this.props.hourHandLength/2,
      paddingVertical: this.props.hourHandWidth,
      paddingLeft: this.props.hourHandLength,
      borderTopLeftRadius: this.props.hourHandCurved ?
                           this.props.hourHandWidth : 0,
      borderBottomLeftRadius: this.props.hourHandCurved ?
                              this.props.hourHandWidth : 0,
      
    }
  }

  minuteHandStyles() {
    return {
      width: 0,
      height: 0,
      position: 'absolute',
      backgroundColor: this.props.minuteHandColor,
      top: this.props.clockSize/2+20,
      left: this.props.clockSize/2+20,
      marginTop: -(this.props.minuteHandLength/2),
      marginHorizontal: -this.props.minuteHandWidth,
      paddingTop: this.props.minuteHandLength,
      paddingHorizontal: this.props.minuteHandWidth,
      borderTopLeftRadius: this.props.minuteHandCurved ?
                           this.props.minuteHandWidth : 0,
      borderTopRightRadius: this.props.minuteHandCurved ?
                            this.props.minuteHandWidth : 0,
    }
  }

  secondHandStyles() {
    return {
      width: 0,
      height: 0,
      position: 'absolute',
      backgroundColor: 'black',
      top: this.props.clockSize/2+20,
      left: this.props.clockSize/2+20,
      marginTop: -(this.props.secondHandLength/2),
      marginHorizontal: -this.props.secondHandWidth,
      paddingTop: this.props.secondHandLength,
      paddingHorizontal: this.props.secondHandWidth,
      borderTopLeftRadius: this.props.secondHandCurved ?
                           this.props.secondHandWidth : 0,
      borderTopRightRadius: this.props.secondHandCurved ?
                            this.props.secondHandWidth : 0,
    }
  }
  render() {
    return (
      <View style={styles.container} >
        <View style={styles.gaugeTop}>

          <AnimatedGaugeProgress
            size={200}
            width={15}
            fill={this.state.data}
            cropDegree={90}
            tintColor="#4682b4"
            backgroundColor="#b0c4de" />

            <View style={[ this.hourHandStyles(),
            {transform:[{ rotate: 2.7*this.state.data-45 + 'deg' },
            {translateX: -(this.props.hourHandOffset +
                           this.props.hourHandLength/2) }]} ]}/>
          
          

         
      {this.state.datas.map((data, index) => {
          return(
            <View style={[ this.secondHandStyles(),
            {transform:[{ rotate: data + 'deg' },
            {translateY: -(this.props.secondHandOffset +
                           this.props.secondHandLength/2+77) }]}]}
          />
          );
        })}

       {this.state.sdatas.map((data, index) => {
          return(
            <View style={[ this.minuteHandStyles(),
            {transform:[{ rotate: data + 'deg' },
            {translateY: -(this.props.minuteHandOffset +
                           this.props.minuteHandLength/2+80) }]}]}
          />
          );
        })}

        </View>
        
        </View>
    );
  }
}

example.defaultProps = {
  clockSize: 200,
  clockBorderWidth: 7.5,
  clockCentreSize: 1,
  clockCentreColor: 'black',
  hourHandColor: 'black',
  hourHandCurved: true,
  hourHandLength: 70,
  hourHandWidth: 2.5,
  hourHandOffset: 0,
  minuteHandColor: '#8f8f8f',
  minuteHandCurved: false,
  minuteHandLength: 5,
  minuteHandWidth: 1,
  minuteHandOffset: 0,
  secondHandColor: 'black',
  secondHandCurved: false,
  secondHandLength: 8,
  secondHandWidth: 2,
  secondHandOffset: 0,
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#ffffff',
  },
  gaugeTop: {
    padding: 20,
    // backgroundColor: '#ff0000'
  },
  rowItem: {
    flex: 0.2,
    alignItems: 'center',
  },
  gaugeBottom: {
    flexDirection: 'row',
    paddingTop: 20,
  },
   pointsDelta: {
    color: '#4c6479',
    fontSize: 50,
    fontWeight: "100"
  },
  pointsDeltaActive: {
    color: '#fff',
  },
  lineStyle: {
    backgroundColor : '#4c6479',
    width : 30,
    height : 1,
    position: 'absolute',
    top: 120,
    left: 120,
  }
});

AppRegistry.registerComponent('example', () => example);
```

**SourceCode:**[gauge](https://github.com/whuhan2013/ReactNativeProject)        
**效果如下**      

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p8.png)

**添加文字**      

```
 {this.state.datas.map((data, index) => {
return(
  <View style={[ this.secondHandStyles(),
  {transform:[{ rotate: data + 'deg' },
  {translateY: -(this.props.secondHandOffset +
                 this.props.secondHandLength/2+77) }]}]}
><Text style={styles.lineStyle}>{index}</Text></View>
);
     })}


 lineStyle: {
 backgroundColor : '#ffffff',
 width : 20,
 height : 20,
 marginLeft: -10,
 fontSize:10
  }
```

![](https://raw.githubusercontent.com/whuhan2013/myImage/master/frontend/p9.png) 

