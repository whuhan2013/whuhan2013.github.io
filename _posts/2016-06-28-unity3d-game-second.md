---
layout: post
title: Unity3D入门(二)
date: 2016-6-28
categories: blog
tags: [Unity3D]
description: Unity3D入门
---

**本文主要包含以下内容** 

1. 2D贴图和帧动画   
2. Unity游戏脚本的生命周期


### 2D贴图和帧动画

2D贴图就好比是在屏幕中绘制了一张静态图片。                   
  帧动画的实现原理就是使用若干张静态图片以一定的时间一帧一帧地在屏幕中切换播放，好比在屏幕中预先设定一个显示动画的区域，然后将图片在这个显示区域中频繁地切换播放。由于绘制的图片有规律地切换播放，给人们带来了视觉的假象，感觉就像播放动画一样。

**帧动画绘制原理：**

首先需要在屏幕中设定一个显示该动画的显示区域，然后将动画中的每一帧图片按照固定的时间在这个区域按顺序切换，继而实现动画的播放。

这里我们使用程序将动画资源存储在动画数组当中，然后设定动画的刷新时间，每次刷新动画时将原有显示区域中绘制下一帧图片，到了最后一帧则从第一帧重新开始。
    

**加载贴图代码** 

```
//贴图
private var texSingle : Texture2D;
//贴图数组
private var texAll : Object[] ;


function OnGUI() 
{
  if(GUI.Button(Rect(0,10,100,50),"加载一张贴图"))
  {
    if(texSingle == null)
    {
      //加载贴图
      texSingle =  Resources.Load("single/1");
    }
  }
  
  if(GUI.Button(Rect(0,130,100,50),"加载一组贴图"))
  {
    if(texAll == null)
    {
      //加载所有贴图
      texAll = Resources.LoadAll("Textures");
    }
  }

  //绘制贴图
  if(texSingle != null)
  {
    //绘制一张贴图
    GUI.DrawTexture(Rect(110,10,120,120), texSingle, ScaleMode.StretchToFill, true, 0);
  }
  
  if(texAll != null)
  {
    for (var i = 0; i < texAll.length; i++) {
      //循环遍历绘制贴图
      GUI.DrawTexture(Rect(110 + i * 120,130,120,120), texAll[i], ScaleMode.StretchToFill, true, 0);
    }
  }

}
```


**帧动画绘制**

```
//动画数组
private var anim: Object[] ;
//帧序列
private var nowFram : int;
//动画帧的总数
private var mFrameCount : int;
//限制一秒多少帧
private var fps : float = 1;
//限制帧的时间 
private var time : float = 0;

function Start()
{
  //得到帧动画中的所有图片资源
  anim = Resources.LoadAll("Textures");
    //得到该动画共有多少帧
  mFrameCount = anim.Length;
  
}

function OnGUI() 
{
  //绘制帧动画
  DrawAnimation(anim,Rect(100,100,32,48));
}


function  DrawAnimation(tex:Object[] , rect : Rect)
{
      //绘制动画信息
    GUILayout.Label("当前动画播放:第"+nowFram+"帧");
    //绘制当前帧
    GUI.DrawTexture(rect, tex[nowFram], ScaleMode.StretchToFill, true, 0);
    //计算限制帧时间
    time += Time.deltaTime;
     //超过限制帧则切换图片
     if(time >= 1.0 / fps){
           //帧序列切换
           nowFram++;
           //限制帧清空
           time = 0;
           //超过帧动画总数从第0帧开始
           if(nowFram >= mFrameCount)
           {
            nowFram = 0;
           }
        } 
}
```


**人物移动实例** 

```
//动画数组
private var animUp: Object[] ;
private var animDown: Object[] ;
private var animLeft: Object[] ;
private var animRight: Object[] ;
//地图贴图
private var map : Texture2D;
//当前人物动画
private var tex : Object[];
//人物X坐标
private var x:int;
//人物Y坐标
private var y:int;
//帧序列
private var nowFram : int;
//动画帧的总数
private var mFrameCount : int;
//限制一秒多少帧
private var fps : float = 10;
//限制帧的时间 
private var time : float = 0;

function Start()
{

  //得到帧动画中的所有图片资源
  animUp = Resources.LoadAll("up");
  animDown = Resources.LoadAll("down");
  animLeft = Resources.LoadAll("left");
  animRight = Resources.LoadAll("right");
    //得到地图资源
    map = Resources.Load("map/map");
    //设置默认动画
    tex  = animUp;
}

function OnGUI() 
{
  //绘制贴图
  GUI.DrawTexture(Rect(0,0,Screen.width,Screen.height), map, ScaleMode.StretchToFill, true, 0);
  
  //绘制帧动画
  DrawAnimation(tex,Rect(x,y,32,48));

    //点击按钮移动人物
  if(GUILayout.RepeatButton("向上"))
  {
    y-=2;
    tex = animUp;
  }
  if(GUILayout.RepeatButton("向下"))
  {
    y+=2;
    tex = animDown;
  }
  if(GUILayout.RepeatButton("向左"))
  {
    x-=2;
    tex = animLeft;
  }
  if(GUILayout.RepeatButton("向右"))
  {
    x+=2;
    tex = animRight;
  }
}


function  DrawAnimation(tex : Object[] , rect : Rect)
{
    //绘制当前帧
    GUI.DrawTexture(rect, tex[nowFram], ScaleMode.StretchToFill, true, 0);
    //计算限制帧时间
    time += Time.deltaTime;
     //超过限制帧则切换图片
     if(time >= 1.0 / fps){
           //帧序列切换
           nowFram++;
           //限制帧清空
           time = 0;
           //超过帧动画总数从第0帧开始
           if(nowFram >= tex.Length)
           {
            nowFram = 0;
           }
        } 
}
```



### Unity游戏脚本的生命周期


Unity游戏脚本在整个游戏开发过程中可以说是关键要素，游戏对象之间任何逻辑的判断都需要脚本来完成。如果说游戏贴图、模型资源的好坏决定一个游戏的视觉品味，那么脚本将直接决定这个游戏的内在质量，决定这个游戏好玩与否。游戏脚本与其他游戏组件用法相同，必须绑定在游戏对象中才能执行它的生命周期。



- function Update（）：正常更新，用于更新逻辑。该方法每帧都会由系统自动调用一次。
- function LateUpdate（）：推迟更新，该方法在Update方法执行后调用，同样每一帧都会调用。
- function Awake（）：脚本唤醒，次方法为系统执行的第一个方法，用于脚本的初始化，此脚本的 生命周期中只执行一次。
- function Start（）：在Awake（）方法之后，Update（）之前执行，并且只执行一次。
- function OnDestroy（）：销毁当前脚本时调用。
- function OnGUI（）：绘制界面



```
//主角对象
private var hero : GameObject;

//按键是否被按下
private var keyUp : boolean; 
private var keyDown : boolean; 
private var keyLeft : boolean; 
private var keyRight : boolean; 

//记录当前时间
private var time : float;
//限制一秒多少帧
private var fps : float = 10;
//帧序列
private var nowFram : int;
//动画数组
private var animUp: Object[] ;
private var animDown: Object[] ;
private var animLeft: Object[] ;
private var animRight: Object[] ;
private var nowAnim: Object[] ;
private var backAnim: Object[] ;

function Start()
{
  //得到主角对象
  hero = GameObject.Find("hero");
  //得到上下左右四组动画
  animUp = Resources.LoadAll("up");
  animDown = Resources.LoadAll("down");
  animLeft = Resources.LoadAll("left");
  animRight = Resources.LoadAll("right");
  nowAnim = animDown;
  backAnim = animDown;
}

function OnGUI() 
{
  //控制主角移动的按钮
  keyUp = GUILayout.RepeatButton("向上");

  keyDown = GUILayout.RepeatButton("向下");
  
  keyLeft = GUILayout.RepeatButton("向左");

  keyRight = GUILayout.RepeatButton("向右");

}

function FixedUpdate()
{
 
  if(keyUp)
  {
    //向上移动
    SetAnimation(animUp);
    hero.transform.Translate(-Vector3.forward * 0.001f);
    
  }
  
  if(keyDown)
  {
    //向下移动
    SetAnimation(animDown);
    hero.transform.Translate(Vector3.forward * 0.001f);
  
  }
  if(keyLeft)
  {
    //向左移动
    SetAnimation(animLeft);
    hero.transform.Translate(Vector3.right * 0.001f);
    
  }
  
  if(keyRight)
  {
    //向右移动
    SetAnimation(animRight);
    hero.transform.Translate(-Vector3.right * 0.001f);
  
  } 
  //播放动画
  DrawAnimation(nowAnim) ;
}

function  DrawAnimation(tex : Object[])
{

    //计算限制帧的时间
    time += Time.deltaTime;
    //超过限制帧切换贴图
     if(time >= 1.0 / fps){
         //帧序列切换
           nowFram++;
         //限制帧清空
           time = 0;
         //超过帧动画总数从第0帧开始
           if(nowFram >= tex.Length)
           {
            nowFram = 0;
           }
        } 
        //将对应的贴图赋予主角对象
        hero.renderer.material.mainTexture = tex[nowFram]; 
}

function SetAnimation(tex : Object[])
{
    //设置播放动画
    nowAnim = tex;
    if(!backAnim.Equals(nowAnim))
    {
    
      nowFram = 0;
      backAnim = nowAnim;
    }

}

```

