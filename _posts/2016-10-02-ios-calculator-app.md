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


**增加model文件** 


```
import Foundation

class CalculatorBrain {
    private enum Op : CustomStringConvertible{
        case operand(Double)
        case UnaryOperation(String,Double->Double)
        case BinaryOperation(String,(Double,Double)->Double)
        
        var description:String{
            get{
                switch self {
                case .operand(let operand):
                    return "\(operand)"
                case .BinaryOperation(let symbol,_):
                    return symbol
                case .UnaryOperation(let symbol, _):
                    return symbol
                
                }
            }
            
        }
        
    }
    
    private var opstack=[Op]()
    private var knowOps=[String:Op]()
    
    init(){
        func learnOp(op:Op){
            knowOps[op.description]=op
        }
        learnOp(Op.BinaryOperation("×"){$0*$1})
        learnOp(Op.BinaryOperation("÷"){$1/$0})
        learnOp(Op.BinaryOperation("+"){$0+$1})
        learnOp(Op.BinaryOperation("-"){$1-$0})
        learnOp(Op.UnaryOperation("√"){sqrt($0)})
//        knowOps["×"]=Op.BinaryOperation("×"){$0*$1}
//        knowOps["÷"]=Op.BinaryOperation("÷"){$1/$0}
//        knowOps["+"]=Op.BinaryOperation("+"){$0+$1}
//        knowOps["-"]=Op.BinaryOperation("-"){$1-$0}
//        knowOps["√"]=Op.UnaryOperation("√"){sqrt($0)}
    }
    
    private func evaluate(ops:[Op])->(result:Double?,remainOps:[Op]){
        if !ops.isEmpty {
            var remainOps=ops;
            let op=remainOps.removeLast()
            switch op {
            case Op.operand(let operand):
                return(operand,remainOps)
            case Op.UnaryOperation(_, let operation):
                let operandEvalution=evaluate(remainOps)
                if let operand=operandEvalution.result{
                    return(operation(operand),operandEvalution.remainOps)
                }
            case Op.BinaryOperation(_, let operation):
                let operandEvlution1=evaluate(remainOps)
                if let operand1=operandEvlution1.result {
                    let operandEvlution2=evaluate(operandEvlution1.remainOps)
                    if let operand2=operandEvlution2.result {
                        return (operation(operand1,operand2),operandEvlution2.remainOps)
                    }
                }
            
            }
        }
        
        return (nil,ops)
    }
    
    func evaluate()->Double?{
        let (result,remainder)=evaluate(opstack)
        print("\(opstack)=\(result)with\(remainder)left over")
        return result
    }
    
    
    func pushOperand(operand:Double)->Double?{
        opstack.append(Op.operand(operand))
        return evaluate()
    }
    
    func performOperation(symbol:String)->Double?{
        if let operation=knowOps[symbol]{
            opstack.append(operation)
        }
        
        return evaluate()
        
    }
    
}
```


controll修改为

```
import UIKit

class CalculViewController: UIViewController {

    @IBOutlet weak var display: UILabel!
    
    var userIsInTheMiddleOFTypingANumber:Bool=false
    var brain=CalculatorBrain()
    
    @IBAction func appendDigit(sender: UIButton) {
        let digit=sender.currentTitle!
        if userIsInTheMiddleOFTypingANumber {
        display.text=display.text!+digit
        }else{
            display.text=digit
            userIsInTheMiddleOFTypingANumber=true
        }
    }
    //var operandstack:Array<Double>=Array<Double>()
    

    @IBAction func operate(sender: UIButton) {
        
        if userIsInTheMiddleOFTypingANumber {
            enter()
        }
        if let operation=sender.currentTitle{
            if let result=brain.performOperation(operation) {
                displayValue=result
            }else{
                displayValue=0
            }
        }
        
    }
    
    
    
    @IBAction func enter() {
        userIsInTheMiddleOFTypingANumber=false
        if let result=brain.pushOperand(displayValue){
            displayValue=result
        }else{
            displayValue=0
        }
        
        
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
    
    
}
```








