---
layout: post
title: 微信之各种数据格式发送与接收
date: 2016-7-18
categories: blog
tags: [微信开发]
description: 微信开发
---

#### 文本（包括表情）             
文字后台格式:
 
```
<xml>
 <ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName>
 <FromUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></FromUserName>
 <CreateTime>1359028446</CreateTime>
 <MsgType><![CDATA[text]]></MsgType>
 <Content><![CDATA[测试文字]]></Content>
 <MsgId>5836982729904121631</MsgId>
</xml> 
```

表情后台格式
 
```
<xml><ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName>
<FromUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></FromUserName>
<CreateTime>1359044526</CreateTime>
<MsgType><![CDATA[text]]></MsgType>
<Content><![CDATA[/::)/::~/::B/::|/:8-)]]></Content>
<MsgId>5837051792978241864</MsgId>
</xml> 
```

XML格式讲解
 
ToUserName 消息接收方微信号，一般为公众平台账号微信号     
FromUserName 消息发送方微信号      
CreateTime 消息创建时间       
MsgType 消息类型；文本消息为text       
Content 消息内容         
MsgId 消息ID号           


**回复文本消息与表情代码**         

```
 //回复文本消息
    private function transmitText($object, $content)
    {
        if (!isset($content) || empty($content)){
            return "";
        }

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[%s]]></Content>
</xml>";
        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time(), $content);

        return $result;
    }
```

![](http://images.cnitblog.com/blog/340216/201312/24212713-99cd9616dec941c89f5a1dfce7fe34d9.jpg)

#### 图片           

后台格式：
 
```
<xml><ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName>
<FromUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></FromUserName>
<CreateTime>1359028479</CreateTime>
<MsgType><![CDATA[image]]></MsgType>
<PicUrl><![CDATA[http://mmbiz.qpic.cn/mmbiz/L4qjYtOibummHn90t1mnaibYiaR8ljyicF3MW7XX3BLp1qZgUb7CtZ0DxqYFI4uAQH1FWs3hUicpibjF0pOqLEQyDMlg/0]]></PicUrl>
<MsgId>5836982871638042400</MsgId>
<MediaId><![CDATA[PGKsO3LAgbVTsFYO7FGu51KUYa07D0C_Nozz2fn1z6VYtHOsF59PTFl0vagGxkVH]]></MediaId>
</xml>
```

 
XML格式讲解
 
```
ToUserName 消息接收方微信号，一般为公众平台账号微信号
FromUserName 消息发送方微信号
CreateTime 消息创建时间
MsgType 消息类型；图片消息为image
PicUrl 图片链接地址，可以用HTTP GET获取
MsgId 消息ID号
```

**代码实现**     

```
 //回复图片消息
    private function transmitImage($object, $imageArray)
    {
        $itemTpl = "<Image>
        <MediaId><![CDATA[%s]]></MediaId>
    </Image>";

        $item_str = sprintf($itemTpl, $imageArray['MediaId']);

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[image]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }
```

![](http://images.cnitblog.com/blog/340216/201312/24212902-120227ed8c2f481ea40f74f387167ccd.jpg)

#### 语音           

后台格式：
 
```
<xml>
    <ToUserName><![CDATA[gh_d035bb259cf5]]></ToUserName>
    <FromUserName><![CDATA[owEUGj4BW8yeWRvyEERiVGKwAF1Q]]></FromUserName>
    <CreateTime>1364883809</CreateTime>
    <MsgType><![CDATA[voice]]></MsgType>
    <MediaId><![CDATA[JfmCezZ3Cwp0FwUvMADwwhvp-XScuvpictubpw0c6ALyA8tj3HLU4PoXzMpIY72P]]></MediaId>
    <Format><![CDATA[amr]]></Format>
    <MsgId>5862131322594912688</MsgId>
</xml> 
```

XML格式讲解
 
```
ToUserName 消息接收方微信号，一般为公众平台账号微信号
FromUserName 消息发送方微信号
CreateTime 消息创建时间
MsgType 消息类型；语音消息为voice
MediaId 媒体ID
Format 语音格式，这里为amr
MsgId 消息ID号 
```

附：AMR接口简介        
全称Adaptive Multi-Rate，主要用于移动设备的音频，压缩比比较大，但相对其他的压缩格式质量比较差，由于多用于人声，通话，效果还是很不错的。          


**实现**         

```
 //回复语音消息
    private function transmitVoice($object, $voiceArray)
    {
        $itemTpl = "<Voice>
        <MediaId><![CDATA[%s]]></MediaId>
    </Voice>";

        $item_str = sprintf($itemTpl, $voiceArray['MediaId']);
        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[voice]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }
```

![](http://images.cnitblog.com/blog/340216/201312/24212942-170dd66e27834ff3ac3d525cbbb60e7c.jpg)


#### 视频            

后台格式：
 
```
xml><ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName>
<FromUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></FromUserName>
<CreateTime>1359028186</CreateTime>
<MsgType><![CDATA[video]]></MsgType>
<MediaId><![CDATA[DBVFRIj29LB2hxuYpc0R6VLyxwgyCHZPbRj_IIs6YaGhutyXUKtFSDcSCPeoqUYr]]></MediaId>
<ThumbMediaId><![CDATA[mxUJ5gcCeesJwx2T9qsk62YzIclCP_HnRdfTQcojlPeT2G9Q3d22UkSLyBFLZ01J]]></ThumbMediaId>
<MsgId>5836981613212624665</MsgId>
</xml> 
```

XML格式讲解
 
```
ToUserName 消息接收方微信号，一般为公众平台账号微信号
FromUserName 消息发送方微信号
CreateTime 消息创建时间
MsgType 消息类型；视频消息为video
MediaId 媒体ID
ThumbMediaId 媒体缩略ID？
MsgId 消息ID号 
```

**实现**           

```
 //回复视频消息
    private function transmitVideo($object, $videoArray)
    {
        $itemTpl = "<Video>
        <MediaId><![CDATA[%s]]></MediaId>
        <ThumbMediaId><![CDATA[%s]]></ThumbMediaId>
        <Title><![CDATA[%s]]></Title>
        <Description><![CDATA[%s]]></Description>
    </Video>";

        $item_str = sprintf($itemTpl, $videoArray['MediaId'], $videoArray['ThumbMediaId'], $videoArray['Title'], $videoArray['Description']);

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[video]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }
```

![](http://images.cnitblog.com/blog/340216/201312/24213106-09fa409d830b447681d148a65f350aa7.jpg)


#### 位置           
后台格式：
 
```
<xml>
<ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName>
<FromUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></FromUserName>
<CreateTime>1359036619</CreateTime>
<MsgType><![CDATA[location]]></MsgType>
<Location_X>22.539968</Location_X>
<Location_Y>113.954980</Location_Y>
<Scale>16</Scale>
<Label><![CDATA[中国广东省深圳市南山区华侨城深南大道9789号 邮政编码: 518057]]></Label>
<MsgId>5837017832671832047</MsgId>
</xml> 
```

XML格式讲解
 
```
 ToUserName 消息接收方微信号，一般为公众平台账号微信号
 FromUserName 消息发送方微信号
 CreateTime 消息创建时间
 MsgType 消息类型，地理位置为location
 Location_X 地理位置纬度
 Location_Y 地理位置经度
 Scale 地图缩放大小
 Label 地理位置信息
 MsgId 消息ID号 
```

**实现**     

```
//接收位置消息
    private function receiveLocation($object)
    {
        $content = "你发送的是位置，经度为：".$object->Location_Y."；纬度为：".$object->Location_X."；缩放级别为：".$object->Scale."；位置为：".$object->Label;
        $result = $this->transmitText($object, $content);
        return $result;
    }
```

![](http://images.cnitblog.com/blog/340216/201312/24213159-d14477a57e0b4aee89adc84d9ed8a05f.jpg)

#### 链接        


后台格式：
 
```
<xml>
<ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName> 
<FromUserName><![CDATA[oIDrpjl2LYdfTAM-oxDgB4XZcnc8]]></FromUserName> 
<CreateTime>1359709372</CreateTime> 
<MsgType><![CDATA[link]]></MsgType> 
<Title><![CDATA[微信公众平台开发者的江湖]]></Title> 
<Description><![CDATA[陈坤的微信公众号这段时间大火，大家..]]></Description> 
<Url><![CDATA[http://israel.duapp.com/web/photo.php]]></Url> 
<MsgId>5839907284805129867</MsgId> 
</xml>  
```

XML格式讲解
 
```
 ToUserName 消息接收方微信号，一般为公众平台账号微信号
 FromUserName 消息发送方微信号
 CreateTime 消息创建时间
 MsgType 消息类型，链接为link
 Title 图文消息标题
 Description 图文消息描述
 Url 点击图文消息跳转链接
 MsgId 消息ID号
```

**实现** 

```
 //接收链接消息
    private function receiveLink($object)
    {
        $content = "你发送的是链接，标题为：".$object->Title."；内容为：".$object->Description."；链接地址为：".$object->Url;
        $result = $this->transmitText($object, $content);
        return $result;
    }
```


#### 图文消息回复  

单条图文消息后台格式：
 
``` 
<xml>
    <ToUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></ToUserName>
    <FromUserName><![CDATA[gh_680bdefc8c5d]]></FromUserName>
    <CreateTime>1359011899</CreateTime>
    <MsgType><![CDATA[news]]></MsgType>
    <Content><![CDATA[]]></Content>
    <ArticleCount>1</ArticleCount>
    <Articles>
        <item>
            <Title><![CDATA[[苹果产品信息查询]]></Title>
            <Description><![CDATA[序列号：USE IMEI NUMBER
IMEI号：358031058974471
设备名称：iPhone 5C
设备颜色：
设备容量：
激活状态：已激活
电话支持：未过期[2014-01-13]
硬件保修：未过期[2014-10-14]
生产工厂：中国]]>
    </Description>
            <PicUrl><![CDATA[http://www.fangbei.org/weixin/weather/icon/banner.jpg]]></PicUrl>
            <Url><![CDATA[]]></Url>
        </item>
    </Articles>
    <FuncFlag>0</FuncFlag>
</xml> 
```

![](http://images.cnitblog.com/blog/340216/201312/24213526-b06250e1b4124e5db281e5d53223d899.jpg)

多条图文消息格式 

```
<xml>
    <ToUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></ToUserName>
    <FromUserName><![CDATA[gh_680bdefc8c5d]]></FromUserName>
    <CreateTime>1359011829</CreateTime>
    <MsgType><![CDATA[news]]></MsgType>
    <Content><![CDATA[]]></Content>
    <ArticleCount>5</ArticleCount>
    <Articles>
        <item>
            <Title><![CDATA[【深圳】天气实况 温度：3℃ 湿度：43﹪ 风速：西南风2级]]></Title>
            <Description><![CDATA[]]></Description>
<PicUrl><![CDATA[http://www.fangbei.org/weixin/weather/icon/banner.jpg]]></PicUrl>
            <Url><![CDATA[]]></Url>
        </item>
        <item>
            <Title><![CDATA[06月24日 周四 2℃~-7℃ 晴 北风3-4级转东南风小于3级]]></Title>
            <Description><![CDATA[]]></Description>
            <PicUrl><![CDATA[http://www.fangbei.org/weixin/weather/icon/d00.gif]]></PicUrl>
            <Url><![CDATA[]]></Url>
        </item>
        <item>
            <Title><![CDATA[06月25日 周五 -1℃~-8℃ 晴 东南风小于3级转东北风3-4级]]></Title>
            <Description><![CDATA[]]></Description>
    <PicUrl><![CDATA[http://www.fangbei.org/weixin/weather/icon/d00.gif]]></PicUrl>
            <Url><![CDATA[]]></Url>
        </item>
        <item>
            <Title><![CDATA[06月26日 周六 -1℃~-7℃ 多云 东北风3-4级转东南风小于3级]]></Title>
            <Description><![CDATA[]]></Description>
<PicUrl><![CDATA[http://www.fangbei.org/weixin/weather/icon/d01.gif]]></PicUrl>
            <Url><![CDATA[]]></Url>
        </item>
        <item>
            <Title><![CDATA[06月27日 周日 0℃~-6℃ 多云 东南风小于3级转东北风3-4级]]></Title>
            <Description><![CDATA[]]></Description>
<PicUrl><![CDATA[http://www.fangbei.org/weixin/weather/icon/d01.gif]]></PicUrl>
            <Url><![CDATA[]]></Url>
        </item>
    </Articles>
    <FuncFlag>0</FuncFlag>
</xml>
```

![](http://images.cnitblog.com/blog/340216/201312/24213651-ebb31b7df0fe4d1aadb44c699ed5c421.jpg)

XML格式讲解
 
```
FromUserName 消息发送方
 ToUserName 消息接收方
 CreateTime 消息创建时间
 MsgType 消息类型，图文消息必须填写news
 Content 消息内容，图文消息可填空
 ArticleCount 图文消息个数，限制为10条以内
 Articles 多条图文消息信息，默认第一个item为大图
  Title 图文消息标题
  Description 图文消息描述
  PicUrl 图片链接，支持JPG、PNG格式，较好的效果为大图640*320，小图80*80
  Url 点击图文消息跳转链接
FuncFlag 星标字段
```

**实现**       

```
 //回复图文消息
    private function transmitNews($object, $newsArray)
    {
        if(!is_array($newsArray)){
            return "";
        }
        $itemTpl = "        <item>
            <Title><![CDATA[%s]]></Title>
            <Description><![CDATA[%s]]></Description>
            <PicUrl><![CDATA[%s]]></PicUrl>
            <Url><![CDATA[%s]]></Url>
        </item>
";
        $item_str = "";
        foreach ($newsArray as $item){
            $item_str .= sprintf($itemTpl, $item['Title'], $item['Description'], $item['PicUrl'], $item['Url']);
        }
        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[news]]></MsgType>
    <ArticleCount>%s</ArticleCount>
    <Articles>
$item_str    </Articles>
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time(), count($newsArray));
        return $result;
    }
```

#### 音乐消息       

后台格式：
 
```
<xml>
    <ToUserName><![CDATA[ollB4jqgdO_cRnVXk_wRnSywgtQ8]]></ToUserName>
    <FromUserName><![CDATA[gh_b629c48b653e]]></FromUserName>
    <CreateTime>1372310544</CreateTime>
    <MsgType><![CDATA[music]]></MsgType>
    <Music>
        <Title><![CDATA[最炫民族风]]></Title>
        <Description><![CDATA[凤凰传奇]]></Description>
        <MusicUrl><![CDATA[http://zj189.cn/zj/download/music/zxmzf.mp3]]></MusicUrl>
        <HQMusicUrl><![CDATA[http://zj189.cn/zj/download/music/zxmzf.mp3]]></HQMusicUrl>
    </Music>
    <FuncFlag>0</FuncFlag>
</xml> 
```

XML格式讲解
 
```
ToUserName     接收方帐号（收到的OpenID）
FromUserName     开发者微信号
CreateTime     消息创建时间
MsgType          消息类型，此处为music
    Title       音乐标题
    Description 音乐描述
    MusicUrl     音乐链接
    HQMusicUrl     高质量音乐链接，WIFI环境优先使用该链接播放音乐
FuncFlag     位0x0001被标志时，星标刚收到的消息。
```

**实现**    

```
//回复音乐消息
    private function transmitMusic($object, $musicArray)
    {
        if(!is_array($musicArray)){
            return "";
        }
        $itemTpl = "<Music>
        <Title><![CDATA[%s]]></Title>
        <Description><![CDATA[%s]]></Description>
        <MusicUrl><![CDATA[%s]]></MusicUrl>
        <HQMusicUrl><![CDATA[%s]]></HQMusicUrl>
    </Music>";

        $item_str = sprintf($itemTpl, $musicArray['Title'], $musicArray['Description'], $musicArray['MusicUrl'], $musicArray['HQMusicUrl']);

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[music]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }
```

#### 事件消息类型
 
目前用户在关注和取消关注，以及点击菜单的时候会自动向公众平台发送事件推送消息：
 
1. 关注事件
 
```
<xml>
    <ToUserName><![CDATA[gh_b629c48b653e]]></ToUserName>
    <FromUserName><![CDATA[ollB4jv7LA3tydjviJp5V9qTU_kA]]></FromUserName>
    <CreateTime>1372307736</CreateTime>
    <MsgType><![CDATA[event]]></MsgType>
    <Event><![CDATA[subscribe]]></Event>
    <EventKey><![CDATA[]]></EventKey>
</xml> 
```

2. 取消关注事件
 
```
<xml>
    <ToUserName><![CDATA[gh_b629c48b653e]]></ToUserName>
    <FromUserName><![CDATA[ollB4jqgdO_cRnVXk_wRnSywgtQ8]]></FromUserName>
    <CreateTime>1372309890</CreateTime>
    <MsgType><![CDATA[event]]></MsgType>
    <Event><![CDATA[unsubscribe]]></Event>
    <EventKey><![CDATA[]]></EventKey>
</xml> 
```

3. 菜单点击事件
 
```
<xml>
    <ToUserName><![CDATA[gh_680bdefc8c5d]]></ToUserName>
    <FromUserName><![CDATA[oIDrpjqASyTPnxRmpS9O_ruZGsfk]]></FromUserName>
    <CreateTime>1377886191</CreateTime>
    <MsgType><![CDATA[event]]></MsgType>
    <Event><![CDATA[CLICK]]></Event>
    <EventKey><![CDATA[天气深圳]]></EventKey>
</xml> 
```

XML格式讲解

```
ToUserName     接收方微信号
FromUserName 发送方微信号，若为普通用户，则是一个OpenID
CreateTime     消息创建时间
MsgType     消息类型，event
Event     事件类型，subscribe(订阅)、unsubscribe(取消订阅)、CLICK(自定义菜单点击事件)
EventKey 事件KEY值，与自定义菜单接口中KEY值对应
```


**全部完整代码**      

```
<?php
/*
    方倍工作室 http://www.fangbei.org/
    CopyRight 2015 All Rights Reserved
*/
header('Content-type:text');

define("TOKEN", "weixin");
$wechatObj = new wechatCallbackapiTest();
if (!isset($_GET['echostr'])) {
    $wechatObj->responseMsg();
}else{
    $wechatObj->valid();
}

class wechatCallbackapiTest
{
    //验证签名
    public function valid()
    {
        $echoStr = $_GET["echostr"];
        $signature = $_GET["signature"];
        $timestamp = $_GET["timestamp"];
        $nonce = $_GET["nonce"];
        $token = TOKEN;
        $tmpArr = array($token, $timestamp, $nonce);
        sort($tmpArr, SORT_STRING);
        $tmpStr = implode($tmpArr);
        $tmpStr = sha1($tmpStr);
        if($tmpStr == $signature){
            echo $echoStr;
            exit;
        }
    }

    //响应消息
    public function responseMsg()
    {
        $postStr = $GLOBALS["HTTP_RAW_POST_DATA"];
        if (!empty($postStr)){
            $this->logger("R \r\n".$postStr);
            $postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
            $RX_TYPE = trim($postObj->MsgType);
            
            //消息类型分离
            switch ($RX_TYPE)
            {
                case "event":
                    $result = $this->receiveEvent($postObj);
                    break;
                case "text":
                    $result = $this->receiveText($postObj);
                    break;
                case "image":
                    $result = $this->receiveImage($postObj);
                    break;
                case "location":
                    $result = $this->receiveLocation($postObj);
                    break;
                case "voice":
                    $result = $this->receiveVoice($postObj);
                    break;
                case "video":
                case "shortvideo":
                    $result = $this->receiveVideo($postObj);
                    break;
                case "link":
                    $result = $this->receiveLink($postObj);
                    break;
                default:
                    $result = "unknown msg type: ".$RX_TYPE;
                    break;
            }
            $this->logger("T \r\n".$result);
            echo $result;
        }else {
            echo "";
            exit;
        }
    }

    //接收事件消息
    private function receiveEvent($object)
    {
        $content = "";
        switch ($object->Event)
        {
            case "subscribe":
                $content = "欢迎使用方倍工作室微信开发入门教程Demo\n请回复以下关键字：文本 表情 单图文 多图文 音乐\n请按住说话 或 点击 + 再分别发送以下内容：语音 图片 小视频 我的收藏 位置";
                $content .= (!empty($object->EventKey))?("\n来自二维码场景 ".str_replace("qrscene_","",$object->EventKey)):"";
                $content .= "\n\n".'<a href="http://m.cnblogs.com/?u=txw1958">技术支持 方倍工作室</a>';
                break;
            case "unsubscribe":
                $content = "取消关注";
                break;
            case "CLICK":
                switch ($object->EventKey)
                {
                    case "COMPANY":
                        $content = array();
                        $content[] = array("Title"=>"方倍工作室", "Description"=>"", "PicUrl"=>"http://discuz.comli.com/weixin/weather/icon/cartoon.jpg", "Url" =>"http://m.cnblogs.com/?u=txw1958");
                        break;
                    default:
                        $content = "点击菜单：".$object->EventKey;
                        break;
                }
                break;
            case "VIEW":
                $content = "跳转链接 ".$object->EventKey;
                break;
            case "SCAN":
                $content = "扫描场景 ".$object->EventKey;
                break;
            case "LOCATION":
                $content = "上传位置：纬度 ".$object->Latitude.";经度 ".$object->Longitude;
                break;
            case "scancode_waitmsg":
                if ($object->ScanCodeInfo->ScanType == "qrcode"){
                    $content = "扫码带提示：类型 二维码 结果：".$object->ScanCodeInfo->ScanResult;
                }else if ($object->ScanCodeInfo->ScanType == "barcode"){
                    $codeinfo = explode(",",strval($object->ScanCodeInfo->ScanResult));
                    $codeValue = $codeinfo[1];
                    $content = "扫码带提示：类型 条形码 结果：".$codeValue;
                }else{
                    $content = "扫码带提示：类型 ".$object->ScanCodeInfo->ScanType." 结果：".$object->ScanCodeInfo->ScanResult;
                }
                break;
            case "scancode_push":
                $content = "扫码推事件";
                break;
            case "pic_sysphoto":
                $content = "系统拍照";
                break;
            case "pic_weixin":
                $content = "相册发图：数量 ".$object->SendPicsInfo->Count;
                break;
            case "pic_photo_or_album":
                $content = "拍照或者相册：数量 ".$object->SendPicsInfo->Count;
                break;
            case "location_select":
                $content = "发送位置：标签 ".$object->SendLocationInfo->Label;
                break;
            default:
                $content = "receive a new event: ".$object->Event;
                break;
        }

        if(is_array($content)){
            $result = $this->transmitNews($object, $content);
        }else{
            $result = $this->transmitText($object, $content);
        }
        return $result;
    }

    //接收文本消息
    private function receiveText($object)
    {
        $keyword = trim($object->Content);
        //多客服人工回复模式
        if (strstr($keyword, "请问在吗") || strstr($keyword, "在线客服")){
            $result = $this->transmitService($object);
            return $result;
        }

        //自动回复模式
        if (strstr($keyword, "文本")){
            $content = "这是个文本消息";
        }else if (strstr($keyword, "表情")){
            $content = "微笑：/::)\n乒乓：/:oo\n中国：".$this->bytes_to_emoji(0x1F1E8).$this->bytes_to_emoji(0x1F1F3)."\n仙人掌：".$this->bytes_to_emoji(0x1F335);
        }else if (strstr($keyword, "单图文")){
            $content = array();
            $content[] = array("Title"=>"单图文标题",  "Description"=>"单图文内容", "PicUrl"=>"http://discuz.comli.com/weixin/weather/icon/cartoon.jpg", "Url" =>"http://m.cnblogs.com/?u=txw1958");
        }else if (strstr($keyword, "图文") || strstr($keyword, "多图文")){
            $content = array();
            $content[] = array("Title"=>"多图文1标题", "Description"=>"", "PicUrl"=>"http://discuz.comli.com/weixin/weather/icon/cartoon.jpg", "Url" =>"http://m.cnblogs.com/?u=txw1958");
            $content[] = array("Title"=>"多图文2标题", "Description"=>"", "PicUrl"=>"http://d.hiphotos.bdimg.com/wisegame/pic/item/f3529822720e0cf3ac9f1ada0846f21fbe09aaa3.jpg", "Url" =>"http://m.cnblogs.com/?u=txw1958");
            $content[] = array("Title"=>"多图文3标题", "Description"=>"", "PicUrl"=>"http://g.hiphotos.bdimg.com/wisegame/pic/item/18cb0a46f21fbe090d338acc6a600c338644adfd.jpg", "Url" =>"http://m.cnblogs.com/?u=txw1958");
        }else if (strstr($keyword, "音乐")){
            $content = array();
            $content = array("Title"=>"最炫民族风", "Description"=>"歌手：凤凰传奇", "MusicUrl"=>"http://mascot-music.stor.sinaapp.com/zxmzf.mp3", "HQMusicUrl"=>"http://mascot-music.stor.sinaapp.com/zxmzf.mp3"); 
        }else{
            $content = date("Y-m-d H:i:s",time())."\n\n".'<a href="http://m.cnblogs.com/?u=txw1958">技术支持 方倍工作室</a>';
        }

        if(is_array($content)){
            if (isset($content[0])){
                $result = $this->transmitNews($object, $content);
            }else if (isset($content['MusicUrl'])){
                $result = $this->transmitMusic($object, $content);
            }
        }else{
            $result = $this->transmitText($object, $content);
        }
        return $result;
    }

    //接收图片消息
    private function receiveImage($object)
    {
        $content = array("MediaId"=>$object->MediaId);
        $result = $this->transmitImage($object, $content);
        return $result;
    }

    //接收位置消息
    private function receiveLocation($object)
    {
        $content = "你发送的是位置，经度为：".$object->Location_Y."；纬度为：".$object->Location_X."；缩放级别为：".$object->Scale."；位置为：".$object->Label;
        $result = $this->transmitText($object, $content);
        return $result;
    }

    //接收语音消息
    private function receiveVoice($object)
    {
        if (isset($object->Recognition) && !empty($object->Recognition)){
            $content = "你刚才说的是：".$object->Recognition;
            $result = $this->transmitText($object, $content);
        }else{
            $content = array("MediaId"=>$object->MediaId);
            $result = $this->transmitVoice($object, $content);
        }
        return $result;
    }

    //接收视频消息
    private function receiveVideo($object)
    {
        $content = "上传视频类型：".$object->MsgType;
        $result = $this->transmitText($object, $content);
        return $result;
    }

    //接收链接消息
    private function receiveLink($object)
    {
        $content = "你发送的是链接，标题为：".$object->Title."；内容为：".$object->Description."；链接地址为：".$object->Url;
        $result = $this->transmitText($object, $content);
        return $result;
    }

    //回复文本消息
    private function transmitText($object, $content)
    {
        if (!isset($content) || empty($content)){
            return "";
        }

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[text]]></MsgType>
    <Content><![CDATA[%s]]></Content>
</xml>";
        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time(), $content);

        return $result;
    }

    //回复图文消息
    private function transmitNews($object, $newsArray)
    {
        if(!is_array($newsArray)){
            return "";
        }
        $itemTpl = "        <item>
            <Title><![CDATA[%s]]></Title>
            <Description><![CDATA[%s]]></Description>
            <PicUrl><![CDATA[%s]]></PicUrl>
            <Url><![CDATA[%s]]></Url>
        </item>
";
        $item_str = "";
        foreach ($newsArray as $item){
            $item_str .= sprintf($itemTpl, $item['Title'], $item['Description'], $item['PicUrl'], $item['Url']);
        }
        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[news]]></MsgType>
    <ArticleCount>%s</ArticleCount>
    <Articles>
$item_str    </Articles>
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time(), count($newsArray));
        return $result;
    }

    //回复音乐消息
    private function transmitMusic($object, $musicArray)
    {
        if(!is_array($musicArray)){
            return "";
        }
        $itemTpl = "<Music>
        <Title><![CDATA[%s]]></Title>
        <Description><![CDATA[%s]]></Description>
        <MusicUrl><![CDATA[%s]]></MusicUrl>
        <HQMusicUrl><![CDATA[%s]]></HQMusicUrl>
    </Music>";

        $item_str = sprintf($itemTpl, $musicArray['Title'], $musicArray['Description'], $musicArray['MusicUrl'], $musicArray['HQMusicUrl']);

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[music]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }

    //回复图片消息
    private function transmitImage($object, $imageArray)
    {
        $itemTpl = "<Image>
        <MediaId><![CDATA[%s]]></MediaId>
    </Image>";

        $item_str = sprintf($itemTpl, $imageArray['MediaId']);

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[image]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }

    //回复语音消息
    private function transmitVoice($object, $voiceArray)
    {
        $itemTpl = "<Voice>
        <MediaId><![CDATA[%s]]></MediaId>
    </Voice>";

        $item_str = sprintf($itemTpl, $voiceArray['MediaId']);
        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[voice]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }

    //回复视频消息
    private function transmitVideo($object, $videoArray)
    {
        $itemTpl = "<Video>
        <MediaId><![CDATA[%s]]></MediaId>
        <ThumbMediaId><![CDATA[%s]]></ThumbMediaId>
        <Title><![CDATA[%s]]></Title>
        <Description><![CDATA[%s]]></Description>
    </Video>";

        $item_str = sprintf($itemTpl, $videoArray['MediaId'], $videoArray['ThumbMediaId'], $videoArray['Title'], $videoArray['Description']);

        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[video]]></MsgType>
    $item_str
</xml>";

        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }

    //回复多客服消息
    private function transmitService($object)
    {
        $xmlTpl = "<xml>
    <ToUserName><![CDATA[%s]]></ToUserName>
    <FromUserName><![CDATA[%s]]></FromUserName>
    <CreateTime>%s</CreateTime>
    <MsgType><![CDATA[transfer_customer_service]]></MsgType>
</xml>";
        $result = sprintf($xmlTpl, $object->FromUserName, $object->ToUserName, time());
        return $result;
    }

    //回复第三方接口消息
    private function relayPart3($url, $rawData)
    {
        $headers = array("Content-Type: text/xml; charset=utf-8");
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_POST, 1);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $rawData);
        $output = curl_exec($ch);
        curl_close($ch);
        return $output;
    }

    //字节转Emoji表情
    function bytes_to_emoji($cp)
    {
        if ($cp > 0x10000){       # 4 bytes
            return chr(0xF0 | (($cp & 0x1C0000) >> 18)).chr(0x80 | (($cp & 0x3F000) >> 12)).chr(0x80 | (($cp & 0xFC0) >> 6)).chr(0x80 | ($cp & 0x3F));
        }else if ($cp > 0x800){   # 3 bytes
            return chr(0xE0 | (($cp & 0xF000) >> 12)).chr(0x80 | (($cp & 0xFC0) >> 6)).chr(0x80 | ($cp & 0x3F));
        }else if ($cp > 0x80){    # 2 bytes
            return chr(0xC0 | (($cp & 0x7C0) >> 6)).chr(0x80 | ($cp & 0x3F));
        }else{                    # 1 byte
            return chr($cp);
        }
    }

    //日志记录
    private function logger($log_content)
    {
        if(isset($_SERVER['HTTP_APPNAME'])){   //SAE
            sae_set_display_errors(false);
            sae_debug($log_content);
            sae_set_display_errors(true);
        }else if($_SERVER['REMOTE_ADDR'] != "127.0.0.1"){ //LOCAL
            $max_size = 1000000;
            $log_filename = "log.xml";
            if(file_exists($log_filename) and (abs(filesize($log_filename)) > $max_size)){unlink($log_filename);}
            file_put_contents($log_filename, date('Y-m-d H:i:s')." ".$log_content."\r\n", FILE_APPEND);
        }
    }
}
?>
```

**参考链接**     
[微信公众平台开发入门教程 - 方倍工作室 - 博客园](http://www.cnblogs.com/txw1958/p/wechat-tutorial.html)

