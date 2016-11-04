---
layout: post
title: IOS自动化测试实现
date: 2016-11-4
categories: blog
tags: [IOS]
description: IOS自动化测试实现
---

ios平台下自动化测试工具概览       

![](http://tmq.qq.com/wp-content/uploads/2016/06/QQ%E6%88%AA%E5%9B%BE20160624145248.png)

表中没有列的内容是稳定性，实践中看来上述除UITesting之外的工具稳定性都相对一般，会出现自身框架导致的各种闪退，以及性能越来越差的问题。
因此iOS平台上除了Monkey测试采用了自动化方式，以及部分性能测试轻度使用了一些自动化工具，大部分功能测试还是依赖于人的操作。

**UITestIng实例**         

```
import Foundation
import XCTest

class UITestDemo_UI_Tests: XCTestCase {
        
    override func setUp() {
        super.setUp()
        
        // Put setup code here. This method is called before the invocation of each test method in the class.
        
        // In UI tests it is usually best to stop immediately when a failure occurs.
        continueAfterFailure = false
        // UI tests must launch the application that they test. Doing this in setup will make sure it happens for each test method.
        XCUIApplication().launch()
    }
    
    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }
    
    func testExample() {
        // Use recording to get started writing UI tests.
        // Use XCTAssert and related functions to verify your tests produce the correct results.
        
        testLoginView()
    }
    
    func testLoginView() {
        let app = XCUIApplication()
        
        // 由于UITextField的id有问题，所以只能通过label的方式遍历元素来读取
        let nameField = app.textFields["name"]
        
        if self.canOperateElement(nameField) {
            nameField.tap()
            nameField.typeText("xiaoming")
        }
        
        let psdField = app.secureTextFields["password"]
       
        if self.canOperateElement(psdField) {
            psdField.tap()
            psdField.typeText("1234321")
        }
        
        // 通过UIButton的预设id来读取对应的按钮
        let loginBtn = app.buttons["Login"]
        if self.canOperateElement(loginBtn) {
            loginBtn.tap()
        }
        
        // 开始一段延时，由于真实的登录是联网请求，所以不能直接获得结果，demo通过延时的方式来模拟联网请求
        let window = app.windows.elementBoundByIndex(0)
        if self.canOperateElement(window) {
            // 延时3秒, 3秒后如果登录成功，则自动进入信息页面，如果登录失败，则弹出警告窗
            window.pressForDuration(3)
        }
        
        // alert的id和labe都用不了，估计还是bug，所以只能通过数量判断
        if app.alerts.count > 0 {
            // 登录失败
            app.alerts.collectionViews.buttons["确定"].tap()
            
            let clear = app.buttons["Clear"]
            if self.canOperateElement(clear) {
                clear.tap()
                
                if self.canOperateElement(nameField) {
                    nameField.tap()
                    nameField.typeText("sun")
                }
                
                if self.canOperateElement(psdField) {
                    psdField.tap()
                    psdField.typeText("111111")
                }
                
                if self.canOperateElement(loginBtn) {
                    loginBtn.tap()
                }
                if self.canOperateElement(window) {
                    // 延时3秒, 3秒后如果登录成功，则自动进入信息页面，如果登录失败，则弹出警告窗
                    window.pressForDuration(3)
                }
                self.loginSuccess()
            }
        } else {
            // 登录成功
            self.loginSuccess()
        }
    }
    
    func loginSuccess() {
        let app = XCUIApplication()
        let window = app.windows.elementAtIndex(0)
        if self.canOperateElement(window) {
            // 延时1秒, push view需要时间
            window.pressForDuration(1)
        }
        self.testInfo()
    }
    
    func testInfo() {
        let app = XCUIApplication()
        let window = app.windows.elementAtIndex(0)
        if self.canOperateElement(window) {
            // 延时2秒, 加载数据需要时间
            window.pressForDuration(2)
        }
        
        let modifyBtn = app.buttons["modify"];
        modifyBtn.tap()
        
        let sexSwitch = app.switches["sex"]
        sexSwitch.tap()
        
        let incrementButton = app.buttons["Increment"]
        incrementButton.tap()
        incrementButton.tap()
        incrementButton.tap()
        app.buttons["Decrement"].tap()
        
        let textView = app.textViews["feeling"]
        textView.tap()
        app.keys["Delete"].tap()
        app.keys["Delete"].tap()
        textView.typeText(" abc ")
        
        // 点击空白区域
        let clearBtn = app.buttons["clearBtn"]
        clearBtn.tap()
        
        // 保存数据
        modifyBtn.tap()
        window.pressForDuration(2)
        
        let messageBtn = app.buttons["message"]
        messageBtn.tap();
        
        // 延时1秒, push view需要时间
        window.pressForDuration(1)
        
        self.testMessage()
    }
    
    func testMessage() {
        let app = XCUIApplication()
        let window = app.windows.elementAtIndex(0)
        if self.canOperateElement(window) {
            // 延时2秒, 加载数据需要时间
            window.pressForDuration(2)
        }
        
        let table = app.tables
        table.childrenMatchingType(.Cell).elementAtIndex(8).tap()
        table.childrenMatchingType(.Cell).elementAtIndex(1).tap()
        
    }
    
    func getFieldWithLbl(label:String) -> XCUIElement? {
        var _:XCUIElement? = nil
        return self.getElementWithLbl(label, type: XCUIElementType.TextField)
    }
    
    func getElementWithLbl(label:String, type:XCUIElementType) -> XCUIElement? {
        let app = XCUIApplication()
        let query = app.descendantsMatchingType(type)
        var result:XCUIElement? = nil
        let num=query.count
        print(num)
        print("****************\n")
        print(label)
        print("***********************\n")
        for i in 0..<num {
            let element:XCUIElement = query.elementBoundByIndex(i)
            let subLabel:String? = element.label;
            if subLabel != nil {
                if subLabel == label {
                    result = element
                }
            }
        }
        return result
    }
    
    func canOperateElement(element:XCUIElement?) -> Bool {
        if element != nil {
            if element!.exists {
                return true
            }
        }
        return false
    }
}
```

**参考**       
[iOS 9 学习系列: UI Testing](http://blog.csdn.net/fish_yan_/article/details/50747861)       
[Xcode7 UI自动化测试详解 带demo UITests](http://www.cocoachina.com/ios/20150925/13566.html)

UITesting存在的问题：只适用于ios9.0以上的机器，以下的机器不能运行        
同时存在不同的模拟器，比如iphon6,iphon6s等，有的可以正常运行，有的却不能等情况，不知道是模拟器的bug，还是程序的bug








