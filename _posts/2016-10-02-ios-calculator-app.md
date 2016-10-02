---
layout: post
title: IOS之计算器实现
date: 2016-10-2
categories: blog
tags: [IOS]
description: IOS之计算器实现
---


利用ios实现计算器app，后期将用mvc结构重构


```
import UIKit

class CalculViewController: UIViewController {

    @IBOutlet weak var display: UILabel!
    
    var userIsInTheMiddleOFTypingANumber:Bool=false
    
    @IBAction func appendDigit(sender: UIButton) {
        let digit=sender.currentTitle!
        if userIsInTheMiddleOFTypingANumber {
        display.text=display.text!+digit
        }else{
            display.text=digit
            userIsInTheMiddleOFTypingANumber=true
        }
    }
    var operandstack:Array<Double>=Array<Double>()
    

    @IBAction func operate(sender: UIButton) {
        let operation=sender.currentTitle!;
        if userIsInTheMiddleOFTypingANumber {
            enter()
        }
        switch operation {
        case "×":performeOperation{$0*$1}
        case "÷":performeOperation{$1/$0}
        case "+":performeOperation{$0+$1}
        case "-":performeOperation{$1-$0}
        case "√":performeOperation{sqrt($0)}
        default:
            break
        }
        
    }
    
//    func multiply(op1:Double,op2:Double) -> Double {
//        return op1*op2;
//    }
    
    func performeOperation(operation:(Double,Double)->Double){
        if operandstack.count>=2 {
            displayValue=operation(operandstack.removeLast(),operandstack.removeLast())
            enter()
        }

    }
    
     private func performeOperation(operation:Double->Double){
        if operandstack.count>=1 {
            displayValue=operation(operandstack.removeLast())
            enter()
        }
        
    }

    
    
    @IBAction func enter() {
        userIsInTheMiddleOFTypingANumber=false
        operandstack.append(displayValue)
        print("operandstack=\(operandstack)")
        
        
    }
    
    var displayValue:Double{
        get{
            return NSNumberFormatter().numberFromString(display.text!)!.doubleValue
        }
        set{
            display.text="\(newValue)"
            userIsInTheMiddleOFTypingANumber=false
        }
    }
```


**知识点如下**  

- 计算型属性的setter与getter
- swift利用函数作为参数  
- swift的重载，详情参见：[swift override](http://stackoverflow.com/questions/29457720/compiler-error-method-with-objective-c-selector-conflicts-with-previous-declara)

**效果如下**

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ios/p1.png)



