---
layout: post
title: React Native实例之BBC新闻客户端
date: 2016-7-8
categories: blog
tags: [React Native]
description: React Basic example
---

看到一个很好的[BBCNews新闻客户端](http://www.wangchenlong.org/2016/05/07/1605/071-rn-bbc-news/),但只有IOS版本，再次将它改造成Android版本。

**主要技术**

- 访问网络请求, 过滤内容, 获取数据.
- 显示多媒体页面, 图片, 视频, 链接等.  

**配置项目**      
初始化项目BBCNews, 直接用npm install,在package.json中写的可能版本冲突               
Html解析库: htmlparser, 时间处理库: moment, 线性梯度库: react-native-linear-gradient, 视频库: react-native-video.


初始化主模块index.android.js, 使用Navigator导航页面, 首页组件Feed模块.           

```
import Feed from './components/Feed.js';

class BBCNew extends Component {
  render() {
    let defaultName='Feed';
      let defaultComponent=Feed;
    return (
<Navigator
            initialRoute={{ name: defaultName,
             component: defaultComponent,
             index: 0 
            }}
            //配置场景
            configureScene=
                {
            (route) => {

//这个是页面之间跳转时候的动画，具体有哪些？可以看这个目录下，有源代码的: node_modules/react-//native/Libraries/CustomComponents/Navigator/NavigatorSceneConfigs.js

            return Navigator.SceneConfigs.VerticalDownSwipeJump;
          }
          }
            renderScene={
            (route, navigator) =>
             {
            let Component = route.component;
            return <Component {...route.params} navigator={navigator} 
                    route={route}
              // Function to call when a new scene should be displayed           
            />
          }
          } />
    );
  }
}
```                     

**新闻列表**   
Feed页面, 主要以列表形式, 即ListView标签, 显示新闻. 未加载完成时, 调用页面加载提示符ActivityIndicatorIOS, 显示动画

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ReactNative/p4.jpg)

```
render() {
  // 未加载完成时, 调用加载页面
  if (!this.state.loaded) {
    return this._renderLoading();
  }
  // ...
}

_renderLoading() {
  return (
    <View style={{flexDirection: 'row', justifyContent: 'center', flex: 1}}>
      <ActivityIndicator
        animating={this.state.isAnimating}
        style={{height: 80}}
        size="large"/>
    </View>
  );
```              

加载完成后, 调用ListView显示页面, renderRow渲染每一行, refreshControl加载页面的过场.

```
return (
  <ListView
    testID={"Feed Screen"}
    dataSource={this.state.dataSource}
    renderRow={this._renderStories.bind(this)}
    style={{backgroundColor: '#eee'}}
    contentInset={{top:0, left:0, bottom: 64, right: 0}}
    scrollEventThrottle={200}
    {...this.props}
    refreshControl={
      <RefreshControl
        refreshing={this.state.isRefreshing}
        onRefresh={this._fetchData.bind(this)}
        tintColor='#BB1919'
        title="Loading..."
        progressBackgroundColor="#FFFF00"
      />}
    />
);
```         

每一行使用Story模块渲染.

```
_renderStories(story) {
  return (
    <Story story={story} navigator={this.props.navigator}/>
  );
}
```    

启动页面的时候, 使用fetch方法加载数据.

```
componentDidMount() {
  this._fetchData();
}
```         

通过访问BBC的网络请求, 异步获取数据. 使用_filterNews过滤需要的数据, 把数据设置入每一行, 修改状态setState, 重新渲染页面.

```
_fetchData() {
  this.setState({isRefreshing: true});
  fetch(`http://trevor-producer-cdn.api.bbci.co.uk/content${this.props.collection || '/cps/news/world'}`)
    .then((response) => response.json())
    .then((responseData) => this._filterNews(responseData.relations))
    .then((newItems) => {
      this.setState({
        dataSource: this.state.dataSource.cloneWithRows(newItems),
        loaded: true,
        isRefreshing: false,
        isAnimating: false
      })
    }).done();
}
```               

then语法是promiser的chaing结构     
                         
> Because the then method returns a Promise, you can easily chain then calls. Values returned from the onFulfilled or onRejected callback functions will be automatically wrapped in a resolved promise.

列表项提供分类显示功能, 点击类别, 可以重新加载所选类型的新闻, 把Feed页面再次添加至导航navigator, 即页面栈.

```
_pressedCollection(collection) {
  this.props.navigator.push({
    component: Feed,
    title: collection.content.name,
    params: {
      collection: collection.content.id,
      navigator: this.props.navigator
    }
  });
}
```          

点击列表项, 跳转至详情页面StoryDetail.

```
_pressedStory(story) {
  this.props.navigator.push({
    component: StoryDetail,
    title: this._truncateTitle(story.content.name),
    params: {story, navigator: this.props.navigator}
  });
}
```

**新闻详情**

主要是解析HTML页面, 加载并显示, 除了文字之外, 会显示图片\视频\超链接等样式. 渲染使用动态元素, 状态state的elements属性.

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ReactNative/p5.jpg)

```
render() {
  if (this.state.loading) {
    return (
      <Text>Loading</Text>
    );
  }
  return this.state.elements;
}
```

页面启动时, 加载数据. 在_fetchStoryData方法中, 进行处理, 使用回调返回数据. 主要内容body与多媒体media通过滚动视图ScrollView的形式显示出来.

```
componentDidMount() {
  this._fetchStoryData(
    // media表示视频或图片.
    (result, media) => {
      const rootElement = result.find(item => {
        return item.name === 'body';
      });

      XMLToReactMap.createReactElementsWithXMLRoot(rootElement, media)
        .then(array => {
          var scroll = React.createElement(ScrollView, {
            contentInset: {top: 0, left: 0, bottom: 64, right: 0},
            style: {flex: 1, flexDirection: 'column', backgroundColor: 'white'},
            accessibilityLabel: "Story Detail"
          }, array);

          this.setState({loading: false, elements: scroll});
        });
    }
  );
}
```          

处理数据, 使用fetch方法, 分离视频与图片, 还有页面, 通过回调cb(callback)的处理返回数据.

```
_fetchStoryData(cb) {
  // 提取数据, 转换JSON格式, 图片过滤, 视频过滤, 组合relations, 解析.
  fetch(`http://trevor-producer-cdn.api.bbci.co.uk/content${this.props.story.content.id}`)
    .then((response) => response.json())
    .then((responseData) => {
      const images = responseData.relations.filter(item => {
        return item.primaryType === 'bbc.mobile.news.image';
      });
      const videos = responseData.relations.filter(item => {
        return item.primaryType === 'bbc.mobile.news.video';
      });
      const relations = {images, videos};
      this._parseXMLBody(responseData.body, (result) => {
        cb(result, relations);
      });
    }).done();
}
```

使用Tautologistics解析dom数据与body数据. DOM, 即Document Object Model, 文件对象模型.

```
_parseXMLBody(body, cb) {
  var handler = new Tautologistics.NodeHtmlParser.DefaultHandler(
    function (error, dom) {
      cb(dom)
    }, {enforceEmptyTags: false, ignoreWhitespace: true});

  var parser = new Tautologistics.NodeHtmlParser.Parser(handler);
  parser.parseComplete(body);
}
```

XML解析类XMLToReactMap比较复杂, 不做过多介绍, 参考源码.


**Codes** 

[react native bbc news client](https://github.com/whuhan2013/BBCNew)

**References** 

[React Native 实例 - BBC新闻客户端 Mystra](http://www.wangchenlong.org/2016/05/07/1605/071-rn-bbc-news/)


