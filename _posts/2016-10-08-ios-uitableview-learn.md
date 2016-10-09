---
layout: post
title: IOS之UITableView
date: 2016-10-8
categories: blog
tags: [IOS]
description: IOS之UITableView
---


ios中的tableView类似于android中的listview，要实现tableview主要包括delegate与datasource两部分

**为了实现dataSource协议,我们需要做什么?**    

1.table分为几个section?func numberOfSectionsInTableView(sender: UITableView) -> Int   
2.在指定的section中有几行?func tableView(sender: UITableView, numberOfRowsInSection: Int) -> Int    
3.给TableView一个UITableViewCell或它的子类     


**UITableViewDelegate**   

1.可通过UITableViewDelegate获取被选中行的信息func tableView(sender: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {  }      
2.可获取detail disclosure被选中的信息func tableView(sender: UITableView, accessoryButtonTappedForRowWithIndexPath: NSIndexPath) {  }         

**如果Model发生变化,如何更新UITableView**

可使用func reloadData()方法更新全部数据,也可使用func reloadRowsAtIndexPaths(indexPaths: [NSIndexPath], withRowAnimation: UITableViewRowAnimation)来更新某一行到某一行之间的数据

**如果Cell的行高发生了变化,如何使其自动适应**                
可将行高设置为UITableViewAutomaticDimension.同时设定estimatedRowHeight获取尺寸的大小.也可让delegate计算每一行的高度func tableView(UITableView,{estimated}heightForRowAtIndexPath: NSIndexPath) -> CGFloat


#### SmashtagDEMO

需要调用twitter api，需要在模拟器中登陆账号，此外原来的twitter工具类是swift1版本的，swift2版本见：[TwitterAPI](https://github.com/Necrocter/cs193p-twitterAPI)

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p14.png?raw=true)


**TweetTableViewController**

```
import UIKit

class TweetTableViewController: UITableViewController, UITextFieldDelegate {

    var tweets = [[Tweet]]()

    // MARK: - ViewControllerLifecycle

    var searchText: String? = "#stanford" {
        didSet {
            lastSuccesfulRequest = nil //每次搜索值改变了以后,将上次的搜索设为nil.因为本次搜索的内同与上次搜索的内容不同,本次的搜索内容与上次的搜索内容无关,在此搜索时需要重新载入请求
            searchTextField.text = searchText
            tweets.removeAll() //当searchText被重新输入新值时,清空推文,并重载tableView,并刷新
            tableView.reloadData()
            refresh()
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        //自动设置tableView的高度
        tableView.estimatedRowHeight = tableView.rowHeight
        tableView.rowHeight = UITableViewAutomaticDimension
        refresh()

    }


    //实现下拉刷新
    var lastSuccesfulRequest: TwitterRequest?

    var nextRequestToAttempt: TwitterRequest? {
        if lastSuccesfulRequest == nil {//如果上次请求未实现,则在进行一次请求
            if searchText != nil {
                return TwitterRequest(search: searchText! , count: 100)
            } else {
                return nil
            }
        }else { //如果上次请求已经实现,则载入新的信息
            return lastSuccesfulRequest!.requestForNewer
        }
    }


    func refresh() { //调用UIRefreshControl,进行刷新
        if refreshControl != nil {
            refreshControl?.beginRefreshing()
        }
        refresh(refreshControl!)
    }


    @IBAction func refresh(sender: UIRefreshControl) {
        if searchText != nil {
            if let request = nextRequestToAttempt {//向Twitter提交请求
                request.fetchTweets { (newTweets) -> Void in //fetchTweets是异步API,多线程API,故下面的UI操作要返回主线程
                    dispatch_async(dispatch_get_main_queue()) { () -> Void in
                        if newTweets.count > 0 {//如果搜索到了新的推文
                            self.lastSuccesfulRequest = request
                            self.tweets.insert(newTweets, atIndex: 0)//将新的推文插入到第一个section中
                            self.tableView.reloadData()//重载tableView
                            sender.endRefreshing()//下拉刷新结束
                        }
                    }
                }
            }

        } else {
            sender.endRefreshing()
        }

    }


    //设置搜索栏
    @IBOutlet weak var searchTextField: UITextField! {
        didSet {
            searchTextField.delegate = self
            searchTextField.text = searchText
        }
    }

    //点击键盘上的return后进行搜索,并且隐藏键盘
    func textFieldShouldReturn(textField: UITextField) -> Bool {
        if textField == searchTextField {
            textField.resignFirstResponder()
            searchText = textField.text
        }
        return true
    }

    // MARK: - UITableViewDataSource

    override func numberOfSectionsInTableView(tableView: UITableView) -> Int {
        return tweets.count
    }

    override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return tweets[section].count
    }
    // 设置Cell的原型
    private struct Storyboaed {
        static let CellReuseIndetifier = "Tweet"
    }
    override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCellWithIdentifier(Storyboaed.CellReuseIndetifier, forIndexPath: indexPath) as! TweetTableViewCell

        // Configure the cell...
        cell.tweet = tweets[indexPath.section][indexPath.row]
        return cell
    }

}
```

**TweetTableViewCell**   

```
import UIKit

class TweetTableViewCell: UITableViewCell {

    var tweet: Tweet? {
        didSet {
            updateUI()
        }
    }

    @IBOutlet weak var tweetProfileImageView: UIImageView!

    @IBOutlet weak var tweetScreenNameLabel: UILabel!

    @IBOutlet weak var tweetTextLabel: UILabel!

    func updateUI() {

        tweetTextLabel?.attributedText = nil
        tweetScreenNameLabel?.text = nil
        tweetProfileImageView?.image = nil



        if let tweet = self.tweet {
            tweetTextLabel?.text = tweet.text
            if tweetTextLabel?.text != nil {
                for _ in tweet.media {
                    tweetTextLabel.text! += " 📷"
                }
            }

            tweetScreenNameLabel?.text = "\(tweet.user)"

            if let profileImageURL = tweet.user.profileImageURL {
                if let imageData = NSData(contentsOfURL: profileImageURL) {
                    tweetProfileImageView?.image = UIImage(data: imageData)
                }
            }
        }
    }

}
```


**问题**  

1. Transport security has blocked a cleartext HTTP        
解决方法参见：[stackoverflow](http://stackoverflow.com/questions/31254725/transport-security-has-blocked-a-cleartext-http)


**源码:**[smashtag](https://github.com/whuhan2013/IOSProject/tree/master/SmashTag)

![](https://github.com/whuhan2013/ImageRepertory/blob/master/ios/p15.png?raw=true)







