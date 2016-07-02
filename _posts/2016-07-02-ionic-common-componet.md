---
layout: post
title: Ionic常用组件
date: 2016-7-2
categories: blog
tags: [Ionic]
description: Ionic common componet
---


**This papper mainly includes the following contents** 

1. Action Sheets    
2. Alerts
3. Badges

### Action Sheets 

Action Sheets slide up from the bottom edge of the device screen, and display a set of options with the ability to confirm or cancel an action. Action Sheets can sometimes be used as an alternative to menus, however, they should not be used for navigation.

The Action Sheet always appears above any other components on the page, and must be dismissed in order to interact with the underlying content. When it is triggered, the rest of the page darkens to give more focus to the Action Sheet options.

**Basic Usage** 

```
import {Component} from '@angular/core';
import {Platform, ActionSheet, NavController} from 'ionic-angular';


@Component({
  templateUrl: 'build/pages/test/test.html'
})
export class Test {
  constructor(public platform: Platform, public nav: NavController) { }

  openMenu() {
    let actionSheet = ActionSheet.create({
      title: 'Albums',
      cssClass: 'action-sheets-basic-page',
      buttons: [
        {
          text: 'Delete',
          role: 'destructive',
          icon: !this.platform.is('ios') ? 'trash' : null,
          handler: () => {
            console.log('Delete clicked');
          }
        },
        {
          text: 'Share',
          icon: !this.platform.is('ios') ? 'share' : null,
          handler: () => {
            console.log('Share clicked');
          }
        },
        {
          text: 'Play',
          icon: !this.platform.is('ios') ? 'arrow-dropright-circle' : null,
          handler: () => {
            console.log('Play clicked');
          }
        },
        {
          text: 'Favorite',
          icon: !this.platform.is('ios') ? 'heart-outline' : null,
          handler: () => {
            console.log('Favorite clicked');
          }
        },
        {
          text: 'Cancel',
          role: 'cancel', // will always sort to be on the bottom
          icon: !this.platform.is('ios') ? 'close' : null,
          handler: () => {
            console.log('Cancel clicked');
          }
        }
      ]
    });

    this.nav.present(actionSheet);
  }
}
```

**Effect**

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ionic2.jpg)


### Alerts  

Alerts are a great way to offer the user the ability to choose a specific action or list of actions. They also can provide the user with important information, or require them to make a decision (or multiple decisions).

From a UI perspective, you can think of Alerts as a type of “floating” modal, that covers only a portion of the screen. This means Alerts should only be used for quick actions like password verification, small app notifications, or quick options. More in depth user flows should be reserved for full screen ​Modals​.

Alerts are quite flexible, and can easily be customized. Check out the API docs for more information.


**different kindes of alerts** 

- Basic Alerts
- Prompt Alerts
- Confirmation Alerts
- Radio Alerts
- Checkbox Alerts   


**Confirmation Alerts**  

Confirmation Alerts are used when it is required that the user explicitly confirms a particular choice before progressing forward in the app. A common example of the Confirmation Alert is checking to make sure a user wants to delete or remove a contact from their address book.

```
import {Component} from '@angular/core';
import {Alert, NavController} from 'ionic-angular';


@Component({
    templateUrl: 'build/pages/test/test.html'
})
export class Test {

  constructor(public nav: NavController) { }

  doConfirm() {
    let confirm = Alert.create({
      title: 'Use this lightsaber?',
      message: 'Do you agree to use this lightsaber to do good across the intergalactic galaxy?',
      buttons: [
        {
          text: 'Disagree',
          handler: () => {
            console.log('Disagree clicked');
          }
        },
        {
          text: 'Agree',
          handler: () => {
            console.log('Agree clicked');
          }
        }
      ]
    });
    this.nav.present(confirm);
  }

}
```


**effects** 

<div style="float:left;border:solid 1px 000;margin:2px;"><img src="https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ionic3.jpg" alt="screenshot" title="screenshot" width="250" height="436" ></div>
<div style="float:left;border:solid 1px 000;margin:2px;"><img src="https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ionic4.png" alt="screenshot" title="screenshot" width="250" height="436" ></div>
<div style="float:left;border:solid 1px 000;margin:2px;"><img src="https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ionic5.png" alt="screenshot" title="screenshot" width="250" height="436" ></div>
<div style="float:left;border:solid 1px 000;margin:2px;"><img src="https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ionic6.png" alt="screenshot" title="screenshot" width="250" height="436" ></div>
<div style="clear:both;"></div>


### Badges