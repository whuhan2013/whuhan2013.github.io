---
layout: post
title: Android自动化测试
date: 2016-9-23
categories: blog
tags: [android]
description: android
---

#### 配置与引入

使用android官方提供的android自动化测试框架uiautomator进行测试

首先配置build.gradle

```
dependencies {
    androidTestCompile 'com.android.support:support-annotations:24.0.0'
    androidTestCompile 'com.android.support.test:runner:0.5'
    androidTestCompile 'com.android.support.test:rules:0.5'
    // Optional -- Hamcrest library
    androidTestCompile 'org.hamcrest:hamcrest-library:1.3'
    // Optional -- UI testing with Espresso
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
    // Optional -- UI testing with UI Automator
    androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.2'
}
```

同时要配置defaultConfig

```
android {
    defaultConfig {
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}
```

#### 利用工具探测ui

Before designing your test, inspect the UI components that are visible on the device. To ensure that your UI Automator tests can access these components, check that these components have visible text labels, android:contentDescription values, or both.

The uiautomatorviewer tool provides a convenient visual interface to inspect the layout hierarchy and view the properties of UI components that are visible on the foreground of the device. This information lets you create more fine-grained tests using UI Automator. For example, you can create a UI selector that matches a specific visible property.

To launch the uiautomatorviewer tool:

1. Launch the target app on a physical device.
2. Connect the device to your development machine.
3. Open a terminal window and navigate to the <android-sdk>/tools/ directory.
4. Run the tool with this command:
$ uiautomatorviewer


To view the UI properties for your application:

1. In the uiautomatorviewer interface, click the Device Screenshot button.
2. Hover over the snapshot in the left-hand panel to see the UI components identified by the uiautomatorviewertool. The properties are listed in the lower right-hand panel and the layout hierarchy in the upper right-hand panel.
3. Optionally, click on the Toggle NAF Nodes button to see UI components that are non-accessible to UI Automator. Only limited information may be available for these components.

#### Create a UI Automator Test Class

The UiDevice object is the primary way you access and manipulate the state of the device. In your tests, you can call UiDevice methods to check for the state of various properties, such as current orientation or display size. Your test can use the UiDevice object to perform device-level actions, such as forcing the device into a specific rotation, pressing D-pad hardware buttons, and pressing the Home and Menu buttons.

```
@RunWith(AndroidJUnit4.class)
@SdkSuppress(minSdkVersion = 18)
public class RunnerTest {
    UiDevice device;
    @Before
    public void setUp(){
        device=UiDevice.getInstance(InstrumentationRegistry.getInstrumentation());
    }

    @Test
    public void testCreateTicket()
    {
        UiObject2 objAdd=device.findObject(By.desc("创建工单"));
        objAdd.clickAndWait(Until.newWindow(),5000);

        UiObject2 objItem=device.findObject(By.text("NancyCustomer1"));
        objItem.clickAndWait(Until.newWindow(),5000);

        this.selectAssets();

        this.selectUsers();

        this.inputTaskDes();

        this.submitTicket();
    }

    public void selectAssets()
    {
        UiObject2 objAsset=device.findObject(By.text("资产范围"));
        objAsset.clickAndWait(Until.newWindow(),5000);

        this.sleepToWait(3000);

        UiObject2 objBuild=device.findObject(By.text("楼宇BADGOOD"));
        objBuild.clickAndWait(Until.newWindow(),5000);//scrollFinished

        this.sleepToWait(3000);

        UiObject2 objPanel=device.findObject(By.text("qq"));
        objPanel.click();

        this.sleepToWait(1000);
        UiObject2 objSaveAsset=device.findObject(By.text("完成"));
        objSaveAsset.clickAndWait(Until.newWindow(),5000);
    }

    public void selectUsers()
    {
        UiObject2 objAsset=device.findObject(By.text("执行人"));
        objAsset.clickAndWait(Until.newWindow(),5000);

        UiObject2 objBuild=device.findObject(By.text("1234"));
        objBuild.click();//scrollFinished

        UiObject2 objPanel=device.findObject(By.text("abcnew"));
        objPanel.click();

        UiObject2 objSaveAsset=device.findObject(By.text("完成"));
        objSaveAsset.clickAndWait(Until.newWindow(),5000);
    }

    public void inputTaskDes()
    {
        UiObject2 objAsset=device.findObject(By.text("工单任务"));
        objAsset.clickAndWait(Until.newWindow(),5000);

        UiObject2 objInput=device.findObject(By.clazz("android.widget.EditText"));
        objInput.setText("This is robot create ticket.");

        this.sleepToWait(1000);

        UiObject2 objSaveAsset=device.findObject(By.text("完成"));
        objSaveAsset.clickAndWait(Until.newWindow(),5000);
    }

    public void submitTicket()
    {
        this.sleepToWait(2000);
        UiObject2 objSaveAsset=device.findObject(By.text("保存"));
        objSaveAsset.clickAndWait(Until.newWindow(),5000);
    }

    private void sleepToWait(int seconds){
        try {
            Thread.sleep(seconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```




