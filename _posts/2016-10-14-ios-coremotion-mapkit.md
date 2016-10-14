---
layout: post
title: IOS之CorMotion与MapKit
date: 2016-10-14
categories: blog
tags: [IOS]
description: IOS之CorMotion与MapKit
---


#### Application Lifecycle and Core Motion

- Notification
	+ For Model (or global) to Controller communication
	+ NSnotificationCenter，类似一个观察者模式，注册并获取变动时的通知
- Application Lifecycle
	+ Not running, Inactive, Active, Background, Suspended
	+ application(UIApplication, didFinishLauchingWithOptions: [NSObject: AnyObject])
	+ 用来知道自己的 app 是在什么情况下被打开的（不只是用户点击这一种）
- UIApplication
	+ Shared instance
		- manages all global behavior
		- never need to subclass it
		- It delegates everything you need to be involved in to its UIApplicationDelegate
- Info.plist
	+ Many of your application’s settings are in Info.plist
- Capabilities
	+ Some features require enabling (iCloud, Game Center, etc)
	+ Switch on in Capabilities tab (Inside your Project Settings)
- Airdrop 需要先注册，参考 Trak 的 info.plist
- Core Motion
	+ API to access motion sensing hardware on your device
	+ Primary inputs: Accelerometer, Gyro, Magnetometer
	+ Class used to get this input is CMMotionManager
	+ Usage
		- Check to see what hardware is available
		- Start the sampling going and poll the motion manager for the latest sample it have
		- …or…
		- check to see what hardware is available
		- Set the rate at which you want data to be reported from the hardware
		- Register a closure (and a queue to run it on) to call each time a sample is taken
	+ Checking availability of hardware sensor
		- var {accelerometer, gyro, magnetometer, deviceMotion}Available: Bool
	+ Starting the hardware sensors collecting data
		- you only need to do this if you are going to poll for data
		- func start{Accelerometer, Gyro, Magnetometer, DeviceMotion}Updates()
	+ Is the hardware currently collection data?
		- var {accelerometer,gyro,magnetometer,deviceMotion}Active: Bool
	+ Stop the hardware collecting data
		- func stop{Accelerometer,Gyro,Magnetometer,DeviceMotion}Updates()
- CMDeviceMotion
	+ 各种传感器的组合



#### Core Location and MapKit  

- Core Location
	+ Framework for managing location and heading
		- No user-interface
	+ Basic object is CLLocation
		- Properties: coordinate, altitude, horizontal/verticallAccuracy, timestamp, speed, course
- The more accuracy you request, the more battery will be used
- CLLocationManager
	+ Check if the hardware you are on/user supports the kind of location updating you want
	+ Create a CLLocationManager instance and set the delegate to receive updates
	+ Configure the manager according to what kind of location updating you want
	+ Start the manager monitoring for location changes
- Kinds of location monitoring
	+ Accuracy-based continual updates
	+ Updates only when “significant” changes in location occur
	+ Region-based updates
	+ Heading monitoring
- Asking CLLocationManager what your hardware can do
	+ class func authorizationStatus() -> CLAuthorizationStatus // Authorized, Denied or Restricted
	+ class func locationServicesEnabled() -> Bool // user enabled (or not) locations for your app
- You must add an Info.plist entry
	+ NSLocationWhenInUseUsageDescription
	+ NSLocationAlwaysUsageDescription
- Error reporting to the delegate
	+ func locationManager(CLLocationManager, didFailWithError: NSError)
	+ Not always a fatal thing, so pay attention to this delegate method
- 也可以进行后台更新
- Map Kit
	+ UI, can have annotations on it
	+ Each annotation is a coordinate, a title and a subtitle
	+ Annotation can have a callout. It appears when the annotation view is clicked
	+ import MapKit
	+ 地图部分也是玩法很多的，需要仔细研究研究
- MKLocalSearch
	+ Can search by natural language strings asynchronously
- MKDirections, MKRoute, MKPolyline
- Overlays, MKOverlayView
- callout 只能代码配合 storyboard 实现，因为 storyboard 压根不会显示这个 callout


**trax Demo**        

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ios/p20.png)

源码：[Trax swift2版本，修改了不少错误](https://github.com/Tsymlov/CS193P-Trax)
参考：[Swift 千金方 2 - Stanford CS193p 学习笔记](http://wdxtub.com/2015/12/18/swift-tricks-2/)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ios/p21.png)



