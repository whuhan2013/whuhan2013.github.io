---
layout: post
title: Ionic常用组件(二)
date: 2016-7-3
categories: blog
tags: [Ionic]
description: Ionic common componet
---


**This papper mainly includes the following contents** 

1. Checkbox  
2. DateTime
3. Gesture   
4. Grid
5. Icon


### Checkbox

A checkbox is an input component that holds a boolean value. Checkboxes are no different than HTML checkbox inputs. However, like other Ionic components, checkboxes are styled differently on each platform. Use the checked attribute to set the default value, and the disabled attribute to disable the user from changing the value.

**Basic Usage**

```
<ion-item>
  <ion-label>Daenerys Targaryen</ion-label>
  <ion-checkbox dark checked="true"></ion-checkbox>
</ion-item>

<ion-item>
  <ion-label>Arya Stark</ion-label>
  <ion-checkbox disabled="true"></ion-checkbox>
</ion-item>
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic9.png)


### DateTime 

The DateTime component is used to present an interface which makes it easy for users to select dates and times. The DateTime component is similar to the native <input type="datetime-local"> element, however, Ionic’s DateTime component makes it easy to display the date and time in a preferred format, and manage the datetime values.


**Basic Usage**

```
<ion-item>
  <ion-label>Start Time</ion-label>
  <ion-datetime displayFormat="h:mm A" pickerFormat="h mm A" [(ngModel)]="event.timeStarts"></ion-datetime>
</ion-item>
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic10.png)


### Gesture 

You can target specific gestures to trigger specific functionality. Built on top of Hammer.js, you can bind to tap, press, pan, swipe, rotate, and pinch.

```
<ion-header>
  <ion-navbar>
    <ion-title>Events</ion-title>
  </ion-navbar>
</ion-header>

<ion-content>

  <ion-card (tap)="tapEvent($event)">
    <ion-item>
      Tapped: {{tap}} times
    </ion-item>
  </ion-card>

  <ion-card (press)="pressEvent($event)">
    <ion-item>
      Pressed: {{press}} times
    </ion-item>
  </ion-card>

  <ion-card (pan)="panEvent($event)">
    <ion-item>
      Panned: {{pan}} times
    </ion-item>
  </ion-card>

  <ion-card (swipe)="swipeEvent($event)">
    <ion-item>
      Swiped: {{swipe}} times
    </ion-item>
  </ion-card>

</ion-content>
```

```

import {Component} from '@angular/core';

@Component({
  templateUrl: './build/pages/gestures/basic/template.html'
})
export class BasicPage {

  public press: number = 0;
  public pan: number = 0;
  public swipe: number = 0;
  public tap: number = 0;
  constructor() {

  }
  pressEvent(e) {
    this.press++
  }
  panEvent(e) {
    this.pan++
  }
  swipeEvent(e) {
    this.swipe++
  }
  tapEvent(e) {
    this.tap++
  }

}
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic11.png)


### Grid


Ionic’s grid system is based on flexbox, a CSS feature supported by all devices that Ionic supports. The grid is composed of three units — grid, rows and columns. Columns will expand to fill their row, and will resize to fit additional columns.

Rows have no padding or margin applied to them. If that is needed, ion-grid can be added to provide additional padding.

In order to set width use width attribute on ion-col tag.



```
<ion-grid>
  <ion-row>
    <ion-col width-10>This column will take 10% of space</ion-col>
  </ion-row>
</ion-grid>
```

Use the offset attribute on a column to set its percent offset from the left (eg: offset-25). To change how columns in a row align vertically, add the center or baseline attribute to an <ion-row>.

Use the wrap attribute on an <ion-row> element to allow items in that row to wrap. This applies the flex-wrap: wrap; style to the <ion-row> element.

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic12.png)


### Icon 

Ionic comes with the same 700+ Ionicons icons we’ve all come to know and love.


**Basic Usage**

To use an icon, just add the Icon’s CSS class to your element:

```
<ion-icon name="heart"></ion-icon>

```


**Active / Inactive Icons**

All icons have both active and inactive states. Active icons are typically full and thick, where as inactive icons are outlined and thin. Set the isActive attribute to true or false to change the state of the icon. Icons will default to active if a value is not specified.

```
<ion-icon name="heart"></ion-icon>                    <!-- active -->
<ion-icon name="heart" isActive="false"></ion-icon>  <!-- inactive -->
```


**Platform Specific Icons:**

Many icons have both Material Design and iOS versions. Ionic will automatically use the correct version based on the platform.

However, if you want more control, you can explicitly set the icon to use for each platform. Use the md (material design) and ios attributes to specify a platform specific icon:

```
<ion-icon ios="logo-apple" md="logo-android"></ion-icon>
```

**Variable Icons:**

To set an icon using a variable:

```
<ion-icon [name]="myIcon"></ion-icon>

export class MyFirstPage {
  // use the home icon
  myIcon: string = "home";
}
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/Ionic/ionic13.png)



