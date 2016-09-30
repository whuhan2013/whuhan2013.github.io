---
layout: post
title: IOS开发第一个app
date: 2016-9-30
categories: blog
tags: [IOS]
description: IOS
---


本文主要记录了从零开始开发第一个ios程序的全过程，主要包含以下内容：

- Build a Basic UI
- Connect the UI to Code
- Work with View Controllers
- Implement a Custom Control
- Define Your Data Model
- Create a Table View
- Implement Navigation
- Implement Edit and Delete Behavior
- Persist Data



#### Build a Basic UI

This lesson gets you familiar with Xcode, the tool you use to write apps. You’ll become familiar with the structure of a project in Xcode and learn how to navigate between and use basic project components. Throughout the lesson, you’ll start making a simple user interface (UI) for the FoodTracker app and view it in Simulator. When you’re finished, your app will look something like this:

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/2_sim_finalUI_2x.png)

this part you will do the follow things

- Create a project in Xcode
- Identify the function of key files that are created with an Xcode project template
- Open and switch between files in a project
- Run an app in Simulator
- Add, move, and resize UI elements in a storyboard
- Edit the attributes of UI elements in a storyboard using the Attributes inspector
- View and rearrange UI elements using the outline view
- Preview a storyboard UI using the Preview assistant editor
- Lay out a UI that automatically adapts to the user’s device size using Auto Layout

#### Connect the UI to Code

In this lesson, you’ll connect the basic UI of the FoodTracker app to code and define some actions a user can perform on that UI. When you’re finished, your app will look something like this:

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/3_sim_finalUI_2x.png)

this part you will do the follow things

- Explain the relationship between a scene in a storyboard and the underlying view controller
- Create outlet and action connections between UI elements in a storyboard and source code
- Process user input from a text field and display the result in the UI
- Make a class conform to a protocol
- Understand the delegation pattern
- Follow the target-action pattern when designing app architecture



#### Work with View Controllers

In this lesson, you’ll continue to work on the UI for the meal scene in the FoodTracker app. You’ll rearrange the existing UI elements and work with an image picker to add a photo to the UI. When you’re finished, your app will look something like this:

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/4_sim_finalUI_2x.png)

this part you will do the follow things

- Understand the view controller life cycle and when its callbacks occur, such as viewDidLoad, viewWillAppear and viewDidAppear 
- Pass data between view controllers
- Dismiss a view controller
- Use gesture recognizers as an additional level of generating events
- Anticipate object behavior based on the UIView/UIControl class hierarchy
- Use the asset catalog to add image assets to a project


#### Implement a Custom Control

In this lesson, you’ll implement a rating control for the FoodTracker app. When you’re finished, your app will look something like this:

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/5_sim_finalUI_2x.png)

this part you will do the follow things

- Create and associate custom source code files with elements in a storyboard
- Define a custom class
- Implement an initializer on a custom class
- Use UIView as a container
- Understand how to display views programmatically


#### Define Your Data Model

In this lesson, you’ll define and test a data model for the FoodTracker app. A data model represents the structure of information in an app.

this part you will do the follow things

- Create a data model
- Write failable initializers on a custom class
- Demonstrate a conceptual understanding of the difference between failable and nonfailable initializers
- Test a data model by writing and running unit tests


#### Create a Table View


In this lesson, you create the main screen of the FoodTracker app. You’ll create a second, table view-based scene, that lists the user’s meals. You’ll design custom table cells to display each meal, which look like this:

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/7_sim_tablecellUI_2x.png)

this part you will do the follow things

- Create a second storyboard scene
- Understand the key components of a table view
- Create and design a custom table view cell
- Understand the roles of table view delegates and data sources
- Use an array to store and work with data
- Display dynamic data in a table view



#### Implement Navigation

In this lesson, you use navigation controllers and segues to create the navigation flow of the FoodTracker app. At the end of the lesson, you’ll have a complete navigation scheme and interaction flow for the app. When you’re finished, your app will look something like this:

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/8_sim_navbar_2x.png)

this part you will do the follow things

- Embed an existing view controller within a navigation controller in a storyboard
- Create segues between view controllers
- Edit the attributes of a segue in a storyboard using the Attributes inspector
- Pass data between view controllers using the prepareForSegue(_:sender:) method
- Perform an unwind segue
- Use stack views to create robust, flexible layouts


#### Implement Edit and Delete Behavior

In this lesson, you focus on adding behavior that allows the user to edit and delete meals in the FoodTracker app.

![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/9_sim_deletebehavior_2x.png)

this part you will do the follow things

- Differentiate between push and modal navigation
- Dismiss view controllers based on their presentation style
- Understand when to use different type cast operators for downcasting
- Leverage optional binding to check for complex conditions
- Use segue identifiers to determine which segue is occurring



#### Persist Data

This lesson is focused on saving a meal list across FoodTracker app sessions. Data persistence is one of the most important and common problems in iOS app development. iOS has many persistent data storage solutions; in this lesson, you’ll use NSCoding as the data persistence mechanism in the FoodTracker app. NSCoding is a protocol that enables a lightweight solution for archiving objects and other structures. Archived objects can be stored on disk and retrieved at a later time.

this part you will do the follow things

- Create a structure
- Understand the difference between static properties and instance properties
- Use the NSCoding protocol to read and write data


![](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/Art/8_sim_navbar_2x.png)

**source code:**[Foodtract](https://github.com/whuhan2013/IOSProject/tree/master/FoodTracker%E5%89%AF%E6%9C%AC)

**Reference:**[start developing ios app](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/)


















