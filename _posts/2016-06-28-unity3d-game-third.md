---
layout: post
title: Unity3D入门(三)
date: 2016-6-28
categories: blog
tags: [Unity3D]
description: Unity3D入门
---

**本文主要包含以下内容** 


1. 动态创建游戏对象
2. 通过对象名称获取对象
3. 通过标签获取多个游戏对象
4. 添加组件和修改组件
5. 发送广播与消息   
6. 对象的克隆与复制
7. 脚本组件  
8. 小地图制作

### 动态创建游戏对象

```
function OnGUI()
{

  if(GUILayout.Button("创建立方体",GUILayout.Height(50)))
  {
    //设置该模型默认为立方体
    var objCube = GameObject.CreatePrimitive(PrimitiveType.Cube);
    //给此对象添加一个刚体用于整理感应
    objCube.AddComponent(Rigidbody);
    //设置这个游戏对象的名称
    objCube.name="Cube";
    //设置此模型材质的颜色
    objCube.renderer.material.color = Color.blue;
    //设置此模型在坐标
    objCube.transform.position = new Vector3(0.0f,10.0f,0.0f);
  }
  
  if(GUILayout.Button("创建球体",GUILayout.Height(50)))
  {
    //设置该模型默认为球体
    var objSphere = GameObject.CreatePrimitive(PrimitiveType.Sphere);
    //给此对象添加一个刚体用于整理感应
    objSphere.AddComponent(Rigidbody);
    //设置这个游戏对象的名称
    objSphere.name="Sphere";
    //设置此模型材质的颜色
    objSphere.renderer.material.color = Color.red;
    //设置此模型在坐标
    objSphere.transform.position = new Vector3(0.0f,10.0f,0.0f);
  }

}
```


### 通过对象名称获取对象

```
//立方体对象
private var objCube : GameObject;
//球体对象
private var objSphere : GameObject;
//是否旋转立方体
private var isCubeRoate  = false;
//是否旋转球体
private var isSphereRoate  = false;
//按钮提示信息
private var CubeInfo: String = "旋转立方体";
private var SphereInfo: String = "旋转球体";

function Start () 
{
  //获取游戏对象
  objCube = GameObject.Find("Cube");
  objSphere = GameObject.Find("Object/Sphere");
}

function Update()
{
  //用户点击旋转按立方体钮后时时旋转模型
  if(isCubeRoate)
  {
    //当立方体对象不为null时旋转
    if(objCube)
    {
      objCube.transform.Rotate(0.0f,Time.deltaTime * 200,0.0f);
    }
  }
  
  //用户点击旋转按球体钮后时时旋转模型
  if(isSphereRoate)
  {
    //当球体对象不为null
    if(objSphere)
    {
      objSphere.transform.Rotate(0.0f,Time.deltaTime * 200,0.0f); 
    }
  }
  
}

function OnGUI()
{
  
  //添加按钮用于旋转立方体
  if(GUILayout.Button(CubeInfo,GUILayout.Height(50)))
  {
    if(!isCubeRoate)
    {
      isCubeRoate = true;
      CubeInfo = "停止旋转立方体";
    }else 
    {
      isCubeRoate = false;
      CubeInfo = "旋转立方体";
    }
  }
  
  //添加按钮用于旋转球体
  if(GUILayout.Button(SphereInfo,GUILayout.Height(50)))
  {
    if(!isSphereRoate)
    {
      isSphereRoate = true;
      SphereInfo = "停止旋转球体";
    }else 
    {
      isSphereRoate = false;
      SphereInfo = "旋转球体";
    }
  }
  
  //添加按钮用于销毁游戏对象
  if(GUILayout.Button("立即销毁模型",GUILayout.Height(50)))
  {
    //立即销毁立方体与球体对象
    Destroy (objCube);
    Destroy (objSphere);
  }
}
```  

### 通过标签获取多个游戏对象

```
var count:int;
function Start () 
{

     //得到包含MyTag标签的游戏对象数组
  var objs = GameObject.FindGameObjectsWithTag ("MyTag");
  count = 0;

  //遍历所有游戏对象
  for (var obj in objs)
  {
  
    count++;
    Debug.Log("以"+ obj.tag+"标签为游戏对象的名称 "+ obj.name+"                第"+count+"个");
    
    
  }
  
}
```


### 添加组件和修改组件

```
//游戏对象
private var obj : GameObject;
//渲染器
private var render : Renderer;
//贴图
public var texture : Texture;

function Start()
{
  //获取游戏对象
  obj = GameObject.Find("Cube");
  //给当前对象添加一个脚本组件
  obj.AddComponent("test_4_1");
  //获取该对象的渲染器
  render = obj.GetComponent("Renderer");


}

function OnGUI()
{
  if(GUILayout.Button("添加颜色",GUILayout.Width(100),GUILayout.Height(50)))
  {
    //修改渲染颜色为绿色
    render.material.color = Color.green;
    //避免残留将贴图置空
    render.material.mainTexture = null;
  }
  if(GUILayout.Button("添加贴图",GUILayout.Width(100),GUILayout.Height(50)))
  {
    //避免残留将贴图置空
    render.material=null;
    //添加组件贴图
    render.material.mainTexture = texture;
  }
  
}

function Update () {

}
```



### 发送广播与消息

```
A0:

function Start () {
  
  //gameObject.BroadcastMessage ("ReceiveBroadcastMessage", "A0-----BroadcastMessage()");
  //gameObject.SendMessage ("ReceiveSendMessage", "A0-----SendMessage()");
  gameObject.SendMessageUpwards ("ReceiveSendMessage", "A0-----SendMessageUpwards()");
  
}

function ReceiveBroadcastMessage(str : String){
  Debug.Log("A0----Receive" +str);
}
function ReceiveSendMessage(str : String){
  Debug.Log("A0----Receive" +str);
}
function ReceiveSendMessageUpwards (str : String){
  Debug.Log("A0----Receive"  +str);
}

A1:
function Start () {
  
  //gameObject.BroadcastMessage ("ReceiveBroadcastMessage", "A1-----BroadcastMessage()");
  //gameObject.SendMessage ("ReceiveSendMessage", "A1-----SendMessage()");
  gameObject.SendMessageUpwards ("ReceiveSendMessage", "A1-----SendMessageUpwards()");
  
}

function ReceiveBroadcastMessage(str : String){
  Debug.Log("A1----Receive" +str);
}
function ReceiveSendMessage(str : String){
  Debug.Log("A1----Receive" +str);
}
function ReceiveSendMessageUpwards (str : String){
  Debug.Log("A1----Receive" +str);
}

B0:
function Start () {
  
  //gameObject.BroadcastMessage ("ReceiveBroadcastMessage", "B0-----BroadcastMessage()");
  //gameObject.SendMessage ("ReceiveSendMessage", "B0-----SendMessage()");
  gameObject.SendMessageUpwards ("ReceiveSendMessage", "B0-----SendMessageUpwards()");
  
}

function ReceiveBroadcastMessage(str : String){
  Debug.Log("B0----Receive" +str);
}
function ReceiveSendMessage(str : String){
  Debug.Log("B0----Receive" +str);
}
function ReceiveSendMessageUpwards (str : String){
  Debug.Log("B0----Receive" +str);
}

B1:
function Start () {
  
  //gameObject.BroadcastMessage ("ReceiveBroadcastMessage", "B1-----BroadcastMessage()");
  //gameObject.SendMessage ("ReceiveSendMessage", "B1-----SendMessage()");
  gameObject.SendMessageUpwards ("ReceiveSendMessage", "B1-----SendMessageUpwards()");
  
}

function ReceiveBroadcastMessage(str : String){
  Debug.Log("B1----Receive" +str);
}
function ReceiveSendMessage(str : String){
  Debug.Log("B1----Receive" +str);
}
function ReceiveSendMessageUpwards (str : String){
  Debug.Log("B1----Receive" +str);
}

C0:
function Start () {
  
  //gameObject.BroadcastMessage ("ReceiveBroadcastMessage", "C0-----BroadcastMessage()");
  //gameObject.SendMessage ("ReceiveSendMessage", "C0-----SendMessage()");
  gameObject.SendMessageUpwards ("ReceiveSendMessage", "C0-----SendMessageUpwards()");
  
}

function ReceiveBroadcastMessage(str : String){
  Debug.Log("C0----Receive" +str);
}
function ReceiveSendMessage(str : String){
  Debug.Log("C0----Receive" +str);
}
function ReceiveSendMessageUpwards (str : String){
  Debug.Log("C0----Receive" +str);
}

C1:
function Start () {
  
  //gameObject.BroadcastMessage ("ReceiveBroadcastMessage", "C1-----BroadcastMessage()");
  //gameObject.SendMessage ("ReceiveSendMessage", "C1-----SendMessage()");
  gameObject.SendMessageUpwards ("ReceiveSendMessage", "C1-----SendMessageUpwards()");
  
}

function ReceiveBroadcastMessage(str : String){
  Debug.Log("C1----Receive" +str);
}
function ReceiveSendMessage(str : String){
  Debug.Log("C1----Receive" +str);
}
function ReceiveSendMessageUpwards (str : String){
  Debug.Log("C1----Receive" +str);
}
```


### 对象的克隆与复制

```
//球体对象
var obj : GameObject;

function Start()
{
  //获得球体对象
  obj = GameObject.Find("Cube");
}

function OnGUI()
{
  
  if(GUILayout.Button("开始克隆实例",GUILayout.Height(50))){
  
    //克隆一个obj的实例
    var clone :GameObject = Instantiate(obj, obj.transform.position, obj.transform.rotation);
    Debug.Log(clone.transform.position);
    //5秒后销毁该实例，
    Destroy (clone, 5);
  }
}
```

### 脚本组件

```
//立方体对象
var obj : GameObject;


function Start () {
  //获得立方体对象
  obj = GameObject.Find("Cube");
}

function OnGUI(){

  if(GUILayout.Button("给立方体添加脚本组件",GUILayout.Height(50))){
    //添加cube_script脚本
    if(obj)
    obj.AddComponent("cube_script");
  }
  
  if(GUILayout.Button("删除立方体脚本组件",GUILayout.Height(50))){
    //删除cube_script脚本
    if(obj)
    Destroy (obj.GetComponent ("cube_script"));
  }
  
  if(GUILayout.Button("立即删除立方体对象",GUILayout.Height(50))){
    //删除立方体对象
    if(obj)
    Destroy (obj);
  }
  
  if(GUILayout.Button("5秒后删除立方体对象",GUILayout.Height(50))){
    //5秒后删除立方体对象
    if(obj)
    Destroy (obj,5);
  }
}


cube_script.js

function Start(){
  Debug.Log("脚本添加成功");
}

function OnDestroy (){
  Debug.Log("脚本删除成功");
}
```


### 小地图制作  

```
using UnityEngine;
using System.Collections;

public class Script_04_17 : MonoBehaviour 
{
  
  //大地图地形对象
  GameObject plane;
  //大地图主角对象
  GameObject cube;
  
  //大地图的宽度
  float mapWidth;
  //大地图的高度
  float mapHeight;
  //地图边界的检测数值
  float widthCheck;
  float heightCheck;
  
  //小地图主角的位置
  float mapcube_x =0;
  float mapcube_y = 0;
  
  
  //GUI按钮是否被按下
  bool keyUp;
  bool keyDown;
  bool keyLeft;
  bool keyRight;
  
  //小地图的背景贴图
  public Texture map;
  //小地图的主角贴图
  public Texture map_cube;
  
  void Start()
  {
    //得到大地图对象
    plane = GameObject.Find("Plane");
    //得到大地图主角对象
    cube = GameObject.Find("Cube");
    //得到大地图默认宽度
    float size_x = plane.GetComponent<MeshFilter>().mesh.bounds.size.x;
    //得到大地图宽度的缩放比例
    float scal_x = plane.transform.localScale.x;
    //得到大地图默认高度
    float size_z = plane.GetComponent<MeshFilter>().mesh.bounds.size.z;
    //得到大地图高度缩放地理
    float scal_z = plane.transform.localScale.z;
    
    //原始宽度乘以缩放比例计算出真实宽度
    mapWidth = size_x * scal_x;
    mapHeight = size_z * scal_z;
    
    //越界监测的宽度
    widthCheck = mapWidth / 2;
    heightCheck = mapHeight / 2;
    
    check();
  }
  
  void OnGUI()
  {
    keyUp = GUILayout.RepeatButton("向前移动");

    keyDown = GUILayout.RepeatButton("向后移动");
    
    keyLeft = GUILayout.RepeatButton("向左移动");
    
    keyRight = GUILayout.RepeatButton("向右移动");
    
    //绘制小地图背景
    GUI.DrawTexture(new Rect(Screen.width - map.width,0,map.width,map.height),map);
    //绘制小地图上的“主角”
    GUI.DrawTexture(new Rect(mapcube_x,mapcube_y,map_cube.width,map_cube.height),map_cube);
  }
  
  void FixedUpdate()
  {
    

    if(keyUp)
    {
      //向前移动
      cube.transform.Translate(Vector3.forward * Time.deltaTime *5); 
      check();
  
    }
    
    if(keyDown)
    {
      //向后移动
      cube.transform.Translate(-Vector3.forward * Time.deltaTime *5); 
      check();
    }
    
    if(keyLeft)
    {
      //向左移动
      cube.transform.Translate(-Vector3.right * Time.deltaTime *5);  
      check();
    }
    
    if(keyRight)
    {
      //向右移动
      cube.transform.Translate(Vector3.right * Time.deltaTime *5); 
      check();
    }
   
    
    
    
  }
  
  //越界检测
  void check()
  {
    //得到当前主角在地图中的坐标
    float x = cube.transform.position.x;
    float z = cube.transform.position.z;
    
    //当控制主角超过地图范围时，重新计算主角坐标
    if(x >= widthCheck)
    {
      x = widthCheck;
    }
    if(x <= -widthCheck)
    {
      x = -widthCheck;
    }
    if(z >= heightCheck)
    {
      z = heightCheck;
    }
    if(z <= -heightCheck)
    {
      z = -heightCheck;
    }
    
    cube.transform.position = new Vector3(x,cube.transform.position.y,z);
    
    //根据比例计算小地图“主角”的坐标
    mapcube_x = (map.width/mapWidth * x) + ((map.width / 2) - (map_cube.width/2)) + (Screen.width - map.width);
    mapcube_y =map.height - ((map.height/mapHeight * z) + (map.height / 2));
  }
}
```