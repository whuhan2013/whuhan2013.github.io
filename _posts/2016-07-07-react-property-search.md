---
layout: post
title: React Native实例之房产搜索APP
date: 2016-7-7
categories: blog
tags: [React Native]
description: React Basic example
---

React Native 开发越来越火了，web app也是未来的潮流, 现在react native已经可以完成一些最基本的功能. 通过开发一些简单的应用, 可以更加熟练的掌握 RN 的知识. 在学习的过程，发现了一个房产搜索APP的实例，但只有ios版本，
本文主要是房产搜索APP的android版本实现。

**原Ios版本**     
[React Native 实例 - 房产搜索App Mystra](http://www.wangchenlong.org/2016/04/12/1604/121-rn-property-demo/)

**原版效果**           
![](http://www.wangchenlong.org/2016/04/12/1604/121-rn-property-demo/rn-property-anim.gif)

**主要内容:**      

- 使用Navigator栈跳转页面.       
- 使用fetch请求网络数据.         
- 使用ListView展示列表数据.             


**首页搜索** 

搜索页(SearchPage)包含一个搜索库, 可以使用地址或邮编搜索英国的房产信息.

通过输入框的参数创建网络请求URL, 并把请求发送出去, 获取信息.

```
/**
 * 访问网络服务的Api, 拼接参数
 * 输出: http://api.nestoria.co.uk/api?country=uk&pretty=1&encoding=json&listing_type=buy&action=search_listings&page=1&place_name=London
 *
 * @param key 最后一个参数的键, 用于表示地理位置
 * @param value 最后一个参数的值, 具体位置
 * @param pageNumber page的页面数
 * @returns {string} 网络请求的字符串
 */
function urlForQueryAndPage(key, value, pageNumber) {
  var data = {
    country: 'uk',
    pretty: '1',
    encoding: 'json',
    listing_type: 'buy',
    action: 'search_listings',
    page: pageNumber
  };

  data[key] = value;

  var querystring = Object.keys(data)
    .map(key => key + '=' + encodeURIComponent(data[key]))
    .join('&');
  return 'http://api.nestoria.co.uk/api?' + querystring;
}

class SearchPage extends Component {

  /**
   * 构造器
   * @param props 状态
   */
  constructor(props) {
    super(props);
    this.state = {
      searchString: 'London', // 搜索词
      isLoading: false, // 加载
      message: '' // 消息
    };
  }

  onSearchTextChanged(event) {
    this.setState({searchString: event.nativeEvent.text});
    console.log(this.state.searchString);
  }


 /**
   * 执行网络请求, 下划线表示私有
   * @param query url
   * @private
   */
  _executeQuery(query) {
    console.log(query);
    this.setState({isLoading: true});

    // 网络请求
    fetch(query)
      .then(response => response.json())
      .then(json => this._handleResponse(json.response))
      .catch(error => this.setState({
        isLoading: false,
        message: 'Something bad happened ' + error
      }));
  }

  /**
   * 处理网络请求的回调
   * @param response 请求的返回值
   * @private
   */
  _handleResponse(response) {
    const { navigator } = this.props;
    this.setState({isLoading: false, message: ''});
    if (response.application_response_code.substr(0, 1) === '1') {
      console.log('123');
      console.log('Properties found: ' + response.listings);

      // 使用listings调用结果页面SearchResults
      navigator.push({
        title: '搜索结果',
        component: SearchResults,
        index:1,
        params:{
                    listings:response.listings,
                    mynav:navigator
                    
                }
       
      });
        console.log('456');
    } else {
      this.setState({message: 'Location not recognized; please try again.'});
    }
  }

  /**
   * 查询的点击事件
   */
  onSearchPressed() {
    var query = urlForQueryAndPage('place_name', this.state.searchString, 1);
    this._executeQuery(query);
  }

  render() {
    var spinner = this.state.isLoading ?
      (<ActivityIndicator size='large'/>) : (<View/>);
    return (
      <View style={styles.container}>
        <Text style={styles.description}>
          搜索英国的房产
        </Text>
        <Text style={styles.description}>
          地址(London)/邮编(W1S 3PR)均可
        </Text>
        <View style={styles.flowRight}>
          <TextInput
            style={styles.searchInput}
            value={this.state.searchString}
            onChange={this.onSearchTextChanged.bind(this)} // bind确保使用组件的实例
            placeholder='Search via name or postcode'/>
          <TouchableHighlight
            style={styles.button}
            underlayColor='#99d9f4'
            onPress={this.onSearchPressed.bind(this)}>
            <Text style={styles.buttonText}>Go</Text>
          </TouchableHighlight>
        </View>
        <Image source={require('./resources/house.png')}
               style={styles.image}/>
        {spinner}
        <Text style={styles.description}>
          {this.state.message}
        </Text>
      </View>
    );
  }
}
```  


![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ReactNative/p1.jpg)


**搜索结果** 

把获取的房产信息, 逐行渲染并显示于ListView中. 


```
class SearchResults extends Component {

  /**
   * 构造器, 通过Navigator调用传递参数(passProps)
   * @param props 状态属性
   */
  constructor(props) {
    super(props);
    var dataSource = new ListView.DataSource(
      {rowHasChanged: (r1, r2) => r1.guid!== r2.guid}
    );
    this.state = {
      dataSource: dataSource.cloneWithRows(this.props.listings)
    };
  }

  /**
   * 点击每行, 通过行数选择信息
   * @param propertyGuid 行ID
   */
  rowPressed(propertyGuid) {
    //var property = this.props.listings.filter(prop => prop.guid == propertyGuid)[0];
    var property=this.props.listings[propertyGuid];
    var mynav=this.props.mynav;
    mynav.push({
      title: '房产信息',
      component: PropertyView,
     index:2,
      params:{
                    property:property,
                    //navigator:this.props.navigator
                    mynav2:mynav
                    
                }
    });
  }

  /**
   * 渲染列表视图的每一行
   * @param rowData 行数据
   * @param sectionID 块ID
   * @param rowID 行ID
   * @returns {XML} 页面
   */
  renderRow(rowData, sectionID, rowID) {
    var price = rowData.price_formatted.split(' ')[0];
    return (
      <TouchableHighlight
        onPress={()=>this.rowPressed(rowID)}
        underlayColor='#dddddd'>
        <View style={styles.rowContainer}>
          <Image style={styles.thumb} source={{uri:rowData.img_url}}/>
          <View style={styles.textContainer}>
            <Text style={styles.price}>${price}</Text>
            <Text style={styles.title} numberOfLines={1}>
              {rowData.title}
            </Text>
          </View>
        </View>
      </TouchableHighlight>
    );
  }

  /**
   * 渲染, 每行使用renderRow渲染
   * @returns {XML} 结果页面的布局
   */
  render() {
    return (
      <ListView
        dataSource={this.state.dataSource}
        renderRow={this.renderRow.bind(this)}
        />
    );
  }
}
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ReactNative/p2.jpg)

**房产信息**         
房产信息是单纯显示页面, 显示图片和文字内容.

```
BackAndroid.addEventListener('hardwareBackPress', function() {
  if(_navigator == null){
    return false;
  }
  if(_navigator.getCurrentRoutes().length === 1){
    return false;
  }
  _navigator.pop();
  return true;
});

var _navigator ;
var PropertyView = React.createClass({
  getInitialState: function()
  {
     _navigator = this.props.mynav2;
    return {

    };
  },
   

  render: function() {
    var property = this.props.property; // 由SearchResult传递的搜索结果
    var stats = property.bedroom_number + ' bed ' + property.property_type;
    if (property.bathroom_number) {
      stats += ', ' + property.bathroom_number + ' ' +
        (property.bathroom_number > 1 ? 'bathrooms' : 'bathroom');
    }

    var price = property.price_formatted.split(' ')[0];

    return (
      <View>

      <View style={styles.container}>

        <Image style={styles.image}
               source={{uri: property.img_url}}/>
        <View style={styles.heading}>
          <Text style={styles.price}>${price}</Text>
          <Text style={styles.title}>{property.title}</Text>
          <View style={styles.separator}/>
        </View>
        <Text style={styles.description}>{stats}</Text>
        <Text style={styles.description}>{property.summary}</Text>
      </View>
      </View>
    );
  }
});
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/ReactNative/p3.jpg)


**Codes** 

[房产搜索APP安卓版](https://github.com/whuhan2013/propertysearch)

欢迎大家Follow,Star.

> 本文参考自[wangchenlong](http://www.wangchenlong.org/2016/04/12/1604/121-rn-property-demo/)

OK, that’s all! Enjoy it!
