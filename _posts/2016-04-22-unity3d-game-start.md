---
layout: post
title: Unity3D入门
date: 2016-6-7
categories: blog
tags: [Unity3D]
description: Unity3D入门
---

### Unity3D是一款应用广泛的3D游戏引擎，本文主要介绍unity3D的简单应用，安装过程略过。

 在游戏的整个开发过程中，游戏界面设计占据非常重要的地位。因为游戏启动后，第一个映入眼帘的就是整个游戏UI界面。UI界面主要包括贴图、按钮和高级控件等。
     Unity为开发者提供了一套非常完善的图形化界面引擎，它包括常见的游戏窗口，文本框，输入框，拖动条，按钮，贴图框等，无论是做软件还是做游戏，都可以很方便地使用  

### GUI高级控件  

Label控件，Button控件，TextField控件，ToolBar空间，Slider控件，ScrollView控件等


### 一些特殊的方法

```
Start（）方法：该方法只执行一次，一般放一些初始化相关的代码。本例中：
function Start()
{
    //得到屏幕宽高
    screenWidth = Screen.width;
    screenHeight = Screen.height;
    //得到图片宽高
    imageWidth = imageTexture.width;
    imageHeight = imageTexture.height;
}


OnGUI（）方法，它是界面绘制方法，所有GUI的绘制都需要在这个方法中实现。本例中，
function OnGUI () 
{
  //将文字内容显示在屏幕中
  GUI.Label(Rect(100, 10, 100, 30), str);
  GUI.Label(Rect(100, 40, 100, 30), "当前屏幕宽:" + screenWidth);
  GUI.Label(Rect(100, 80, 100, 30), "当前屏幕高:" + screenHeight);
  //将贴图显示在屏幕中
  GUI.Label(Rect(100, 120, imageWidth, imageHeight),imageTexture);
}
```   

### 对变量的声明  

```
  只有公有变量才可以在编辑器中拖拽对象或者以输入的形式赋值。如本例中的“HelloWorld” 以及图片对象。
   在声明变量时，在变量前方添加public关键字或未添加任何关键字都可以表示该变量为公有变量。本例中，

//接收外部赋值字符串
public var str :String;
//接收外部赋值贴图
var imageTexture : Texture;
//贴图宽度
private var imageWidth : int;
//贴图高度
private var imageHeight :int; 
//当前屏幕高度
private var screenWidth :int;
//当前屏幕宽度
private var screenHeight :int;
```   


### label控件  

```
//接收外部赋值字符串
public var str :String;
//接收外部赋值贴图
var imageTexture : Texture;
//贴图宽度
private var imageWidth : int;
//贴图高度
private var imageHeight :int; 
//当前屏幕高度
private var screenWidth :int;
//当前屏幕宽度
private var screenHeight :int;

function Start()
{
    //得到屏幕宽高
    screenWidth = Screen.width;
  screenHeight = Screen.height;
  //得到图片宽高
  imageWidth = imageTexture.width;
  imageHeight = imageTexture.height;
}

function OnGUI () 
{
  //将文字内容显示在屏幕中
  GUI.Label(Rect(100, 10, 100, 30), str);
  GUI.Label(Rect(100, 40, 100, 30), "当前屏幕宽:" + screenWidth);
  GUI.Label(Rect(100, 80, 100, 30), "当前屏幕高:" + screenHeight);
  //将贴图显示在屏幕中
  GUI.Label(Rect(100, 120, imageWidth, imageHeight),imageTexture);
}
```   


### Button控件   

```
//按钮贴图
var buttonTexture : Texture2D;

//提示信息
private var str : String;

//时间计数器
private var frameTime : int;

function Start()
{
  //初始化赋值
  str = "请您点击按钮";
}

function OnGUI() 
{
  
    //显示提示信息内容
    GUI.Label(Rect(10, 10, Screen.width, 30), str);
  
    if(GUI.Button(Rect(10,50,buttonTexture.width,buttonTexture.height),buttonTexture)){
      //点击按钮修改提示信息
      str = "您点击了图片按钮";
    }
    //设置按钮中文字的颜色
    GUI.color = Color.green;
      //设置按钮的背景色
    GUI.backgroundColor = Color.red;
      
      if (GUI.Button(Rect(10,300,70,30),"文字按钮")){
      //点击按钮修改提示信息
      str = "您点击了文字按钮";
      }
      
     //设置按钮中文字的颜色
    GUI.color = Color.yellow;
      //设置按钮的背景色
    GUI.backgroundColor = Color.black;
      
      if (GUI.RepeatButton(Rect(10,350,100,30),"按钮按下中")){
      
      //点击按钮修改提示信息
      str = "按钮按下中的时间："+ frameTime;
      //时间计数器++
      frameTime++;
      }
      
}
```   

### TextField控件  


```
//用户名
private var editUsername : String;
//密码
private var editPassword : String;
//提示信息
private var editShow : String;

function Start()
{
  editShow = "请您输入正确的用户名与密码";
  editUsername = "请输入用户名";
  editPassword = "请输入密码";
}

function OnGUI () 
{
  
  //显示提示信息内容
  GUI.Label(Rect(10, 10, Screen.width, 30), editShow);
  
  if (GUI.Button(Rect(10,120,100,50),"登录"))
  {
    //点击按钮修改提示信息
    editShow = "您输入的用户名为 ：" + editUsername + " 您输入的密码为："+ editPassword;
  }
  //编辑框提示信息
  GUI.Label(Rect(10, 40, 50, 30), "用户名");
  
  GUI.Label(Rect(10, 80, 50, 30), "密码：");
    
    //获取输入框输入的内容
    editUsername = GUI.TextField (Rect (60, 40, 200, 30), editUsername, 15);
    editPassword = GUI.PasswordField  (Rect (60, 80, 200, 30), editPassword, "*"[0],15);
}

```    


### ToolBar控件  

```
//工具栏选择按钮的ID
private var select : int;

//工具栏显示按钮的字符串
private var barResource : String[];

//选择按钮是否被按下
private var selectToggle0: boolean;
private var selectToggle1: boolean;

function Start()
{
  //初始化
  select = 0;
  barResource = ["第一个工具栏","第二个工具栏","第三个工具栏","第四个工具栏"];
  
  selectToggle0 = false;
  selectToggle1 = false;
}

function OnGUI () 
{
  //备份上一次工具栏选择的ID
  var oldSelect = select;
  //重新计算本次工具栏选择的ID
  select = GUI.Toolbar(Rect (10, 10, barResource.length * 100, 30), select, barResource);
  //如果两次选择的是不同的工具栏，将选择按钮全部释放掉
  if(oldSelect != select){
    selectToggle0 = false;
    selectToggle1 = false;
  }
  
  //根据工具栏选择的ID 显示不同的信息
  switch(select)
  {
  case 0:
    selectToggle0 = GUI.Toggle(Rect(10, 50, 200, 30), selectToggle0, "第一个工具栏单项选择——1");
    selectToggle1 = GUI.Toggle(Rect(10, 80, 200, 30), selectToggle1, "第一个工具栏单项选择——2");
    break;
  case 1:
    selectToggle0 = GUI.Toggle(Rect(10, 50, 200, 30), selectToggle0, "第二个工具栏单项选择——1");
    selectToggle1 = GUI.Toggle(Rect(10, 80, 200, 30), selectToggle1, "第二个工具栏单项选择——2");
    break;  
  case 2:
    selectToggle0 = GUI.Toggle(Rect(10, 50, 200, 30), selectToggle0, "第三个工具栏单项选择——1");
    selectToggle1 = GUI.Toggle(Rect(10, 80, 200, 30), selectToggle1, "第三个工具栏单项选择——2");
      break;
  case 3:
    selectToggle0 = GUI.Toggle(Rect(10, 50, 200, 30), selectToggle0, "第四个工具栏单项选择——1");
    selectToggle1 = GUI.Toggle(Rect(10, 80, 200, 30), selectToggle1, "第四个工具栏单项选择——2");
    break;    
  } 
}
```   

### slide控件  

```
//纵向滑动条数值
var verticalValue : int = 0;

//横向滑动条数值
var horizontalValue : float = 0.0f;

function OnGUI () 
{
  //计算滑动进度
  verticalValue = GUI.VerticalSlider (Rect (25, 25, 30, 100), verticalValue, 100, 0);
  horizontalValue = GUI.HorizontalSlider(Rect (50, 25, 100, 30), horizontalValue, 0.0f, 100.0f);
  //将滑动进度显示在屏幕中
  GUI.Label(Rect(10, 150, Screen.width, 30), "纵向滑动条当前进度: " + verticalValue +"%");
  GUI.Label(Rect(10, 180, Screen.width, 30), "横向滑动条当前进度: " + horizontalValue +"%");
}
```   

### scrollView  

```
//滚动条位置
var scrollPosition : Vector2;


function Start()
{
  //初始化滚动条位置
  scrollPosition[0] = 50;
  scrollPosition[1] = 50;

}

function OnGUI () {
    //设置开始滚动视图
  scrollPosition = GUI.BeginScrollView (Rect (0,0,200,200),scrollPosition, Rect (0, 0, Screen.width, 300),true,true);

  GUI.Label(Rect(100, 40, Screen.width, 30), "测试滚动视图，测试滚动视图，测试滚动视图，测试滚动视图。");

    //设置结束滚动视图
  GUI.EndScrollView ();
  
}
```   


### 群组视图   

 群组视图（GroupView控件）可以将多个视图全部放在一个群组当中。将视图添加进群组当中后，群组中任何视图的坐标都是相对坐标，它是相对群组视图左上角的坐标。
    修改群组视图的坐标都是相对坐标，群组中所有视图的坐标都会跟着修改。推荐使用群组视图来制作游戏界面，因为设备的屏幕尺寸不同，这样做可以避免堆坐标进行多次修改的麻烦。
     GUIContent（）方法：设置提示信息
     GUI.tooltip：可以得到GUIContent中的提示字符串
     GUI.BeginGroup()：创建一个群组视图，必须以GUI.EndGroup()结束群组视图。
      在该区域可以添加任意控件，如果超出该范围，则不予显示。


```
//贴图
var viewTexture0 : Texture2D;
var viewTexture1 : Texture2D;


function OnGUI () 
{

    //开始这个群组
  GUI.BeginGroup(new Rect(300, 50, 200, 400));
  //显示贴图，坐标为相对群组的点(10，50)
  GUI.DrawTexture(Rect(10,0,viewTexture0.width,viewTexture0.height), viewTexture0);
    //标签提示信息
  GUI.Label(Rect(10,260,100,40),"群组视图1");
  //按钮
  GUI.Button(Rect(10,280,100,40),"按钮");
  //结束这个群组
  GUI.EndGroup();


  //开始这个群组
  GUI.BeginGroup(new Rect(600, 50, 500, 400));
  //显示贴图，坐标为相对群组的点(300，0)
  GUI.DrawTexture(Rect(10,20,viewTexture1.width,viewTexture1.height), viewTexture1);
    //标签提示信息
  GUI.Label(Rect(10,280,100,40), "群组视图2");
  //按钮
  GUI.Button(Rect(10,300,100,40),"按钮");
  //结束这个群组
  GUI.EndGroup();

} 
```  


### 窗口  
  在游戏中所有视图都需要依赖窗口来显示，我们可以把窗口理解成视图的父类。即游戏界面可以由若干个窗口组成，窗口又由若干个视图组成。
    窗口中所有控件的坐标均采用相对坐标（相对于窗口左上角的坐标）。


```
//默认窗口位置
private var window0 : Rect = Rect (20, 20, 200, 200);
private var window1 : Rect = Rect (250, 20, 200, 200);
function OnGUI () 
{
  //在这里注册两个窗口
  GUI.Window (0, window0, oneWindow, "第一个窗口");
  GUI.Window (1, window1, twoWindow, "第二个窗口" );
}
//显示窗口1的内容
function oneWindow (windowID : int) {
  
  GUI.Box(Rect(10,50,150,50),"这里窗口的ID是" + windowID);
  if(GUI.Button(Rect(10,120,150,50),"普通按钮"))
  {
    Debug.Log("窗口id = "+windowID+"按钮被点击");
  }
  
}
//显示窗口2的内容
function twoWindow (windowID : int) {
  GUI.Box(Rect(10,50,150,50),"这里窗口的ID是" + windowID);
  if(GUI.Button(Rect(10,120,150,50),"普通按钮"))
  {
    Debug.Log("窗口id = "+windowID+"按钮被点击");
  }
}
```   

### 完成 































