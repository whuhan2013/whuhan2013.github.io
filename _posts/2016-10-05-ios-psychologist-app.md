---
layout: post
title: psychologist app实现
date: 2016-10-5
categories: blog
tags: [IOS]
description: IOS之psychologist
---


**ios笑脸pyschologist实现**    



iOS provides some Controllers whose View is “other MVCs”   

- UITabBarController   
- UISplitViewController 
- UINavigationController
- popover

**psychologist主要功能**   

- 使用UISplistViewController分为master与Detail部分
- 根据不同选项展示不同笑脸,使用segue
- 存储历史选择并用popover显示



**使用UISplistViewController分为master与Detail部分**  

主要是在storyboard上操作,拖动控件即可

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p4.png?raw=true)


**根据不同选项展示不同笑脸**

```
import UIKit

class PsychologistViewController: UIViewController {

    
    
    @IBAction func nothing(sender: UIButton) {
        performSegueWithIdentifier("nothing", sender: nil)
    }
    
    
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        var destination=segue.destinationViewController as? UIViewController
        if let navCon=destination as?UINavigationController{
            destination=navCon.visibleViewController
        }
        if let hvc=destination as? HappinessViewController{
            if let identifer=segue.identifier{
                switch identifer {
                case "sad":
                    hvc.happiness=0
                case "happy":
                    hvc.happiness=100
                case "nothing":
                    hvc.happiness=25
                default:
                    hvc.happiness=50
                }
            }
        }
    }


}
```

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p5.png?raw=true)


**存储历史选择并用popover显示**

使用DiagnosedHappinessViewController继承happinessViewController并存储相关信息

```
import UIKit

class DiagnosedHappinessViewController: HappinessViewController ,UIPopoverPresentationControllerDelegate{
    
    override var happiness: Int{
        didSet{
            diagnosticHistory+=[happiness]
        }
    }
    
    private let defaults=NSUserDefaults.standardUserDefaults()
    
    var diagnosticHistory:[Int]{
        get{
            return defaults.objectForKey(History.DefaultKey) as? [Int] ?? []
        }
        set{
            defaults.setObject(newValue, forKey: History.DefaultKey)
        }
    }
    
    private struct History {
        static let SegueIndentifier="Show Diagnostic History"
        static let DefaultKey="DiagnosedHappinessViewController.History"
    }
    
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        if let identifier=segue.identifier{
            switch identifier {
            case History.SegueIndentifier:
                if let tvc=segue.destinationViewController as? TextViewController{
                    if let ppc=tvc.popoverPresentationController{
                        ppc.delegate=self
                    }
                    
                    tvc.text="\(diagnosticHistory)"
                    
                }
            default:
                break
            }
        }
    }
    
    func adaptivePresentationStyleForPresentationController(controller: UIPresentationController) -> UIModalPresentationStyle {
        return UIModalPresentationStyle.None
    }
    
}
```

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p6.png?raw=true)

**SourceCode:** [Psychologist](https://github.com/whuhan2013/IOSProject/tree/master/Psychologist)





