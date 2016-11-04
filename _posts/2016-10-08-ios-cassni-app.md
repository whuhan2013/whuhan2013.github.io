---
layout: post
title: Cassni app实现
date: 2016-10-8
categories: blog
tags: [IOS]
description: IOS之Cassni
---


ios cassni app主要用到了以下内容:

1. scrollview的使用
2. 闭包的使用
3. 多线程的使用

**scrollview的使用**  

使用scrollview包住原来的view即可，需要控制contensize,如果需要缩放，还需要添加最大与最小缩放范围。

添加ScrollView

1.设置其滑动区域(最大可显示区域)  scrollView. contentSize = CGSize(width: 3000, height:2000)    
2.设置位置及大小 logo.frame = CGRect(x:2700, y:50, width:120, height: 180)    
3.添加 scrollView.addSubview(logo)    

获取正在显示的区域  

利用ScrollView的contentOffset属性即可 let upperLeftOfVisible:CGPoint = scrollView.contentOffset

zoom in 放大/缩小ScrollView也可以进行放大和缩小

1.设置代理,实现viewForZoomingInScrollView方法 func viewForZoomingInScrollView(sender: UIScrollView) -> UIView   
2.设置zoom in的最大值和最小值 ScrollView.minimumZoomScale = 0.5 //最小缩小为原来的1/2   
ScrollView.maxmumZoomScale = 2.0 //最大放大到原来的2倍     
3.利用代码进行缩放设置  

```
var zoomScale: CGFloat 
func setZoomScale(CGFloat, animated: Bool)
func zoomToRect(CGRect, animated: Bool)
```

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p10.png?raw=true)

**闭包的使用**

[Swift闭包详解](http://c.biancheng.net/cpp/html/2285.html)


**多线程的使用**

**Queues**

在iOS里有多重队列,每个队列就相当于一个函数队列,基础的闭包在等待运行.每个队列都有一个自己的线程去运行这些队列、去处理队列里的东西.这造就了多线程环境

Main Queue

1.Main Queue是什么?　　    
　　主队列是一个串行队列,这说明主队列依次从队列里拉出一个函数.它从来不会同一时间运行两个函数.所有的UI活动都必须发生在主队列.

2.如何获得Main Queue

```
let mainQ: dispatch_queue_t = dispatch_get_main_queue()
let mainQ: NSOperationQueue = NSOperationQueue.mainQueue()
//以上两种方法都可获得Main Queue
dispatch_async(notTheMainQueue) {
   //此处跳出主队列,获取其他队列,在此一般添加一些不阻塞UI的代码,如DEMO中的加载URL,已实现程序的顺畅运行
   dispatch_async(dispatch_get_main_queue()) {
   //返回主队列,此时已经完成了上面对URL的加载,这时再来更新UI
   }
}
```

Other Queues   如何获得      

1.向系统申请一个恰当的service来获取队列

```
QOS_CLASS_USER_INTERACTIVE //最高优先级,立即执行
QOS_CLASS_USER_INITIATED //优先级较高,但加载会占用较长时间
QOS_CLASS_UTILITY //较低优先级
QOS_CLASS_BACKGROUND //最低优先级,可在后台进行处理,与用户的操作无关
let qos = Int(以上四个QOS的其中之一.rawValue)
```


2.创建队列

```
let queue = dispatch_get_global_queue(qos,0)
```

3.进行队列的操作,实现代码即可


#### CassiniDemo

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p11.png?raw=true)

```
import UIKit

class ImageViewController: UIViewController,UIScrollViewDelegate {
    
    var imageURL : NSURL?{
        didSet{
            image=nil
            if view.window != nil{
            fetchImage()
            }
        }
    }
    
    func fetchImage(){
        if let url=imageURL{
            spinner?.startAnimating()
            let qos = Int (QOS_CLASS_USER_INTERACTIVE.rawValue)
            dispatch_async(dispatch_get_global_queue(qos, 0), { () -> Void in
                let imageData=NSData(contentsOfURL:url )
                dispatch_async(dispatch_get_main_queue()){
                    if url == self.imageURL{
                        if imageData != nil{
                            self.image = UIImage(data: imageData!)
                        }else{
                            self.image=nil
                        }
                    }
            }
            })
            
        }
    }
    
    func viewForZoomingInScrollView(scrollView: UIScrollView) -> UIView? {
        return imageView
    }
    
    private var imageView=UIImageView()
    
    @IBOutlet weak var scrollView: UIScrollView!{
        didSet{
            scrollView.contentSize = imageView.frame.size
            scrollView.delegate=self
            scrollView.minimumZoomScale=0.03
            scrollView.maximumZoomScale=1.0
        }
    }
    
    @IBOutlet weak var spinner: UIActivityIndicatorView!
    
    
    private var image:UIImage?{
        get{
            return imageView.image
        }
        set{
            imageView.image=newValue
            imageView.sizeToFit()
            scrollView?.contentSize = imageView.frame.size
            spinner?.stopAnimating()
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        scrollView.addSubview(imageView)
        
//        if image == nil{
//            imageURL = DemoURL.Stanford
//        }
    }
    
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        if image == nil{
            fetchImage()
        }
    }
}
```

[source](https://github.com/whuhan2013/IOSProject/tree/master/Cassni)

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p12.png?raw=true)



