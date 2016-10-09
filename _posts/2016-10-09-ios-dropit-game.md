---
layout: post
title: IOS之dropit小游戏
date: 2016-10-9
categories: blog
tags: [IOS]
description: IOS之dropit小游戏
---


**本文主要包括以下内容** 

1. Unwind Segues, Alerts, Timers
2. Dynamic Animation
3. dropit小游戏实例

#### Unwind Segues, Alerts, Timers

**Unwind**      
Unwind Segue不会像其他Segue一样创建一个新的MVC


**Alerts/Action Sheets**       
如何添加Alerts/Action Sheets

1.创建button    
2.调用UIAlertController      
3.addAction     
4.调用presentViewController显示Alerts/Action Sheets    

这次在之前CassiniDEMO中加入了Alerts和Action Sheets.
点进Cassini后点击"Redeploy"弹出Action Sheets.
在Action Sheets中"Orbit Saturn"栏中实现了点击跳出Alerts的功能


```
具体代码如下:

    @IBOutlet weak var redeploy: UIBarButtonItem!
    @IBAction func redeploy(sender: UIBarButtonItem) {
        //创建actionSheet
        let actionSheet = UIAlertController(title: "Redeploy", message: "Issue commands to Cassini'sguidance system.", preferredStyle: UIAlertControllerStyle.ActionSheet)
        //添加Action
        actionSheet.addAction(UIAlertAction(title: "Orbit Saturn", style: UIAlertActionStyle.Default, handler: { (action: UIAlertAction) -> Void in
            //此处实现了,点击Orbit Saturn后跳出Alert
            let alert1 = UIAlertController(title: "Login Required", message: "Please enter your Cassini guidance system...", preferredStyle: UIAlertControllerStyle.Alert)//创建Alert
            //添加Action
            alert1.addAction(UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Cancel, handler: nil))
            alert1.addAction(UIAlertAction(title: "Login", style: UIAlertActionStyle.Default, handler: nil))
            //给Alert添加一个TextField
            alert1.addTextFieldWithConfigurationHandler { (textField) in
                textField.placeholder = "Guidance System Password"
            }

            //显示TextField
            self.presentViewController(alert1, animated: true, completion: nil)

        }))


        actionSheet.addAction(UIAlertAction(title: "Explore Titan", style: UIAlertActionStyle.Default, handler: nil))

        actionSheet.addAction(UIAlertAction(title: "Closeup of Sun", style: UIAlertActionStyle.Destructive, handler: nil))

        actionSheet.addAction(UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Cancel, handler: nil))

        //以下三行代码用于iPad已POPover的形式显示ActionSheets.在iPhone上不需要
        actionSheet.modalPresentationStyle = .Popover
        let ppc = actionSheet.popoverPresentationController
        ppc?.barButtonItem = redeploy

        //显示actionSheet
        presentViewController(actionSheet, animated: true, completion: nil)
    }

```


#### Animation

**iOS中动画的分类**   

1.UIView Animation(视图属性的动画)      
2.Animation of View Controller transitions(viewController转场动画)     
3.Core Animation      
4.Dynamic Animation(基于物理引擎的动画)      

**UIView Animation**

- 在其中我们主要能够使用三种属性,分别为:frame(意味着可移动);transform(意味着可旋转);alpha(意味着可改变透明度)
- 使用:class func animateWithDuration(durationL NSTimeInterval, delay: NSTimerInterval, options: UIViewAnimationOptions, animations: () -> Void, completion: ((finished: Bool) -> Void)?)

**Dynamic Animation**

如何使用Dynamic Animation       
1.创建UIDynamicAnimator实例 var animator = UIDynamicAnimator(referenceView: UIView)      
2.添加UIDynamicBehaviors,如gravity,collisions等 let gravity = UIGravityBehavior()
animation.addBehavior(gravity)                 
3.添加UIDynamicItem let item1: UIDynamicItem = ...// usually a UIView
gravity.addItem(item1)


**Behaviors**      
1.UIGravityBehavior有两个重要属性var angle: CGFloat  //重力的角度     var magnitude: CGFloat  //重力加速度     
2.UIAttachmentBehavior       
3.UICollisionBehavior碰撞效果      
4.UISnapBehavior震荡效果       
5.UIPushBehavior把物体推向特定角度,速度恒定,可食一次性的效果,也可是持续效果      
6.UIDynamicItemBehavior控制对象之间如何交互          

#### DropitDEMO

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ios/p16.png)


```
import UIKit

class dropitViewController: UIViewController,UIDynamicAnimatorDelegate {

    @IBOutlet weak var gameView: BezierPathsView!

    //创建DynamicAnimator
    lazy var animator: UIDynamicAnimator = {
        let lazilyCreatedDynamicAnimator = UIDynamicAnimator(referenceView: self.gameView)
        lazilyCreatedDynamicAnimator.delegate = self
        return lazilyCreatedDynamicAnimator
    }()

    //创建集成了三个Behavior的DropBehavior的实例
    var dropitBehavior = DropBehavior()

    var attachment: UIAttachmentBehavior? {
        willSet {
            animator.removeBehavior(attachment!)
            gameView.setPath(nil, named: PathNames.Attachment)
        }
        didSet {
            if attachment != nil {
                animator.addBehavior(attachment!)
                attachment?.action = { [unowned self] in
                    if let attacheView = self.attachment?.items.first as? UIView {
                        let path = UIBezierPath()
                        path.moveToPoint(self.attachment!.anchorPoint)
                        path.addLineToPoint(attacheView.center)
                        self.gameView.setPath(path, named: PathNames.Attachment)
                    }
                }
            }
        }
    }


    override func viewDidLoad() {
        super.viewDidLoad()
        animator.addBehavior(dropitBehavior)
    }

    struct PathNames {
        static let MiddleBarrier = "Middle Barrier"
        static let Attachment = "Attachment"
    }


    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        let barrierSize = dropSize
        let barrierOrigin = CGPoint(x: gameView.bounds.midX - barrierSize.width / 2, y: gameView.bounds.midY - barrierSize.height / 2)
        //设置圆圈的边界
        let path = UIBezierPath(ovalInRect: CGRect(origin: barrierOrigin, size: barrierSize))
        dropitBehavior.addBarrier(path, named: PathNames.MiddleBarrier)

        gameView.setPath(path, named: PathNames.MiddleBarrier)
    }

    //UIDynamicAnimatorDelegate中的方法,当动画停止时
    func dynamicAnimatorDidPause(animator: UIDynamicAnimator) {
        removeCompletedRow()
    }


    var dropsPerRow = 10
    //设置方块的形状
    var dropSize: CGSize {
        let size = gameView.bounds.width / CGFloat(dropsPerRow)
        return CGSize(width: size, height: size)
    }

    //设置手势
    @IBAction func drop(sender: UITapGestureRecognizer) {
        drop()
    }

    var lastDroppedView: UIView?

    //掉落方法
    func drop() {
        //设置方块的初始位置及大小
        var frame = CGRect(origin: CGPointZero, size: dropSize)
        frame.origin.x = CGFloat.random(dropsPerRow) * dropSize.width

        //创建方块视图,并设置其颜色
        let dropView = UIView(frame: frame)
        dropView.backgroundColor = UIColor.random

        lastDroppedView = dropView

        dropitBehavior.addDrop(dropView)
    }


    @IBAction func grabDrop(sender: UIPanGestureRecognizer) {
        let gesturePoint = sender.locationInView(gameView)

        switch sender.state {
        case .Began:
            if let viewToAttachTo = lastDroppedView {
                attachment = UIAttachmentBehavior(item: viewToAttachTo, attachedToAnchor: gesturePoint)
                lastDroppedView = nil
            }
        case .Changed:
            attachment?.anchorPoint = gesturePoint
        case .Ended:
            attachment = nil
        default: break
        }

    }

    //当满行后消除方块
    func removeCompletedRow() {
        var dropsToRemove = [UIView]()
        var dropFrame = CGRect(x: 0, y: gameView.frame.maxY, width: dropSize.width, height: dropSize.height)

        repeat {
            dropFrame.origin.y -= dropSize.height
            dropFrame.origin.x = 0
            var dropsFound = [UIView]()
            var rowIsComplete = true
            for _ in 0 ..< dropsPerRow {
                if let hitView = gameView.hitTest(CGPoint(x: dropFrame.midX, y: dropFrame.midY), withEvent: nil) {
                    if hitView.superview == gameView {
                        dropsFound.append(hitView)
                    } else {
                        rowIsComplete = false
                    }
                }
                dropFrame.origin.x += dropSize.width            }
            if rowIsComplete {
                dropsToRemove += dropsFound
            }
        } while dropsToRemove.count == 0 && dropFrame.origin.y > 0

        for drop in dropsToRemove {
            dropitBehavior.removeDrop(drop)
        }
    }

}

private extension CGFloat {
    static func random(max:Int) -> CGFloat {
        return CGFloat(arc4random() % UInt32(max))
    }
}

private extension UIColor {
    class var random:UIColor {
        switch arc4random() % 5 {
        case 0: return UIColor.greenColor()
        case 1: return UIColor.blueColor()
        case 2: return UIColor.orangeColor()
        case 3: return UIColor.redColor()
        case 4: return UIColor.purpleColor()
        default: return UIColor.blackColor()
        }
    }
}
```



```
import UIKit

class dropitViewController: UIViewController,UIDynamicAnimatorDelegate {

    @IBOutlet weak var gameView: BezierPathsView!

    //创建DynamicAnimator
    lazy var animator: UIDynamicAnimator = {
        let lazilyCreatedDynamicAnimator = UIDynamicAnimator(referenceView: self.gameView)
        lazilyCreatedDynamicAnimator.delegate = self
        return lazilyCreatedDynamicAnimator
    }()

    //创建集成了三个Behavior的DropBehavior的实例
    var dropitBehavior = DropBehavior()

    var attachment: UIAttachmentBehavior? {
        willSet {
            animator.removeBehavior(attachment!)
            gameView.setPath(nil, named: PathNames.Attachment)
        }
        didSet {
            if attachment != nil {
                animator.addBehavior(attachment!)
                attachment?.action = { [unowned self] in
                    if let attacheView = self.attachment?.items.first as? UIView {
                        let path = UIBezierPath()
                        path.moveToPoint(self.attachment!.anchorPoint)
                        path.addLineToPoint(attacheView.center)
                        self.gameView.setPath(path, named: PathNames.Attachment)
                    }
                }
            }
        }
    }


    override func viewDidLoad() {
        super.viewDidLoad()
        animator.addBehavior(dropitBehavior)
    }

    struct PathNames {
        static let MiddleBarrier = "Middle Barrier"
        static let Attachment = "Attachment"
    }


    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        let barrierSize = dropSize
        let barrierOrigin = CGPoint(x: gameView.bounds.midX - barrierSize.width / 2, y: gameView.bounds.midY - barrierSize.height / 2)
        //设置圆圈的边界
        let path = UIBezierPath(ovalInRect: CGRect(origin: barrierOrigin, size: barrierSize))
        dropitBehavior.addBarrier(path, named: PathNames.MiddleBarrier)

        gameView.setPath(path, named: PathNames.MiddleBarrier)
    }

    //UIDynamicAnimatorDelegate中的方法,当动画停止时
    func dynamicAnimatorDidPause(animator: UIDynamicAnimator) {
        removeCompletedRow()
    }


    var dropsPerRow = 10
    //设置方块的形状
    var dropSize: CGSize {
        let size = gameView.bounds.width / CGFloat(dropsPerRow)
        return CGSize(width: size, height: size)
    }

    //设置手势
    @IBAction func drop(sender: UITapGestureRecognizer) {
        drop()
    }

    var lastDroppedView: UIView?

    //掉落方法
    func drop() {
        //设置方块的初始位置及大小
        var frame = CGRect(origin: CGPointZero, size: dropSize)
        frame.origin.x = CGFloat.random(dropsPerRow) * dropSize.width

        //创建方块视图,并设置其颜色
        let dropView = UIView(frame: frame)
        dropView.backgroundColor = UIColor.random

        lastDroppedView = dropView

        dropitBehavior.addDrop(dropView)
    }


    @IBAction func grabDrop(sender: UIPanGestureRecognizer) {
        let gesturePoint = sender.locationInView(gameView)

        switch sender.state {
        case .Began:
            if let viewToAttachTo = lastDroppedView {
                attachment = UIAttachmentBehavior(item: viewToAttachTo, attachedToAnchor: gesturePoint)
                lastDroppedView = nil
            }
        case .Changed:
            attachment?.anchorPoint = gesturePoint
        case .Ended:
            attachment = nil
        default: break
        }

    }

    //当满行后消除方块
    func removeCompletedRow() {
        var dropsToRemove = [UIView]()
        var dropFrame = CGRect(x: 0, y: gameView.frame.maxY, width: dropSize.width, height: dropSize.height)

        repeat {
            dropFrame.origin.y -= dropSize.height
            dropFrame.origin.x = 0
            var dropsFound = [UIView]()
            var rowIsComplete = true
            for _ in 0 ..< dropsPerRow {
                if let hitView = gameView.hitTest(CGPoint(x: dropFrame.midX, y: dropFrame.midY), withEvent: nil) {
                    if hitView.superview == gameView {
                        dropsFound.append(hitView)
                    } else {
                        rowIsComplete = false
                    }
                }
                dropFrame.origin.x += dropSize.width            }
            if rowIsComplete {
                dropsToRemove += dropsFound
            }
        } while dropsToRemove.count == 0 && dropFrame.origin.y > 0

        for drop in dropsToRemove {
            dropitBehavior.removeDrop(drop)
        }
    }

}

private extension CGFloat {
    static func random(max:Int) -> CGFloat {
        return CGFloat(arc4random() % UInt32(max))
    }
}

private extension UIColor {
    class var random:UIColor {
        switch arc4random() % 5 {
        case 0: return UIColor.greenColor()
        case 1: return UIColor.blueColor()
        case 2: return UIColor.orangeColor()
        case 3: return UIColor.redColor()
        case 4: return UIColor.purpleColor()
        default: return UIColor.blackColor()
        }
    }
}
```



```
import UIKit

class BezierPathsView: UIView {

    private var bezierPath = [String: UIBezierPath]()

    func setPath(path: UIBezierPath?, named name: String) {
        bezierPath[name] = path
        setNeedsDisplay()
    }

    override func drawRect(rect: CGRect) {
        for (_,path) in bezierPath {
            path.stroke()
        }
    }

}
```


**参考链接:**[Stanford CS193p iOS开发课程笔记(十)](http://www.jianshu.com/p/3ac404544928)

**源码：** [Dropit](https://github.com/whuhan2013/IOSProject/tree/master/Dropit)

**效果如下**


![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ios/p17.png)

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ios/p18.png)








