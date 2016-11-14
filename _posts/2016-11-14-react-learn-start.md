---
layout: post
title: React学习入门
date: 2016-11-14
categories: blog
tags: [javascript]
description: React学习入门
---

![](http://img.mukewang.com/581f0f4500014daf12800720.jpg)    

**JSX: JS XML**    
语法糖：糖衣语法，计算机语言中添加的某种语法，对语言的功能没有影响，更方便程序员使用，增加程序的可读性，降低出错的可能性

类form结构，以及自定义html标签，这些统称为react Components，这并不是html标签，只是react Components的实例，通过调用ReactDOM.render(，)方法,第一个参数是需要渲染的react Components，第二个参数是渲染完以后需要呈现的Element位置
自定义的Components通过调用React.createClass()方法创建，参数是JS的object，
eg：
var Hello = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }

});

ReactDOM.render(
  <Hello name="World" />,
  document.getElementById('container')
);
{}表示其中要执行JS表达式的值
this是指当前Components的实例，props表示使用react Components时，在其上添加的属性的集合，key值和定义时候的值相同例如{this.props.name}的值为World，和定义的name值相同

class 在ES6中，是一个关键字，用来声明一个类，在之前的ES5等标准中，是一个保留字，所以，在react中，不能再标签上直接写class，因为其是JS运行环境。声明css样式可以使用className。在react中行内样式不是 用字符串的形式表示，需要用样式对象来表示。eg：style={{color:'red'}},写法符合驼峰标示。