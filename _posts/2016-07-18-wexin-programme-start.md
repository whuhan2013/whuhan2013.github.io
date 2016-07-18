---
layout: post
title: 微信开发入门
date: 2016-7-18
categories: blog
tags: [微信开发]
description: 微信开发
---

**微信开发需要搭建PHP环境**     

参考链接：[Apache2.2与php5.3.5如何整合？如何能够被使用_百度经验](http://jingyan.baidu.com/article/ff42efa911a3a0c19e220297.html)

搭建的过程出现了很多坑，只能小心，重复搭建了，错误原因可能是VC9或VC11环境没有配置的原因，多试试才行。

然后通过基本配置修改微信服务器配置，这样发送的消息就会发送到我们的URL，而不是微信官方的，从而实现自定义微信。

接入微信公众平台开发，开发者需要按照如下步骤完成：

1、填写服务器配置       
2、验证服务器地址的有效性         
3、依据接口文档实现业务逻辑           

**第一步：填写服务器配置**

登录微信公众平台官网后，在公众平台后台管理页面 - 开发者中心页，点击“修改配置”按钮，填写服务器地址（URL）、Token和EncodingAESKey，其中URL是开发者用来接收微信消息和事件的接口URL。Token可由开发者可以任意填写，用作生成签名（该Token会和接口URL中包含的Token进行比对，从而验证安全性）。EncodingAESKey由开发者手动填写或随机生成，将用作消息体加解密密钥。

同时，开发者可选择消息加解密方式：明文模式、兼容模式和安全模式。模式的选择与服务器配置在提交后都会立即生效，请开发者谨慎填写及选择。加解密方式的默认状态为明文模式，选择兼容模式和安全模式需要提前配置好相关加解密代码，详情请参考消息体签名及加解密部分的文档。    

![](http://mp.weixin.qq.com/wiki/static/assets/d2e9528d76c9756ca46f03d4bd93e719.png)

此处的URL为自定义的云应用的IP或域名,而Token在index.php中定义为weixin，由你自己定义，也可以为其他的。EncodingAsKey
不用填，可以直接生成，


**第二步：验证服务器地址的有效性**

开发者提交信息后，微信服务器将发送GET请求到填写的服务器地址URL上，GET请求携带四个参数：

signature	微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。        
timestamp	时间戳         
nonce	随机数         
echostr	随机字符串        

下面的代码实现了输入一个？返回当前时间的功能，其中valid的方法，是用于验证接口的，responseMsg方法用于返回消息  

```
<?php
/*
    方倍工作室 http://www.cnblogs.com/txw1958/
    CopyRight 2013 www.fangbei.org  All Rights Reserved
*/

define("TOKEN", "weixin");
$wechatObj = new wechatCallbackapiTest();
if (isset($_GET['echostr'])) {
    $wechatObj->valid();
}else{
    $wechatObj->responseMsg();
}

class wechatCallbackapiTest
{
    public function valid()
    {
        $echoStr = $_GET["echostr"];
        if($this->checkSignature()){
            header('content-type:text');
            echo $echoStr;
            exit;
        }
    }

    private function checkSignature()
    {
        $signature = $_GET["signature"];
        $timestamp = $_GET["timestamp"];
        $nonce = $_GET["nonce"];

        $token = TOKEN;
        $tmpArr = array($token, $timestamp, $nonce);
        sort($tmpArr, SORT_STRING);
        $tmpStr = implode( $tmpArr );
        $tmpStr = sha1( $tmpStr );

        if( $tmpStr == $signature ){
            return true;
        }else{
            return false;
        }
    }

    public function responseMsg()
    {
        $postStr = $GLOBALS["HTTP_RAW_POST_DATA"];

        if (!empty($postStr)){
            $postObj = simplexml_load_string($postStr, 'SimpleXMLElement', LIBXML_NOCDATA);
            $fromUsername = $postObj->FromUserName;
            $toUsername = $postObj->ToUserName;
            $keyword = trim($postObj->Content);
            $time = time();
            $textTpl = "<xml>
                        <ToUserName><![CDATA[%s]]></ToUserName>
                        <FromUserName><![CDATA[%s]]></FromUserName>
                        <CreateTime>%s</CreateTime>
                        <MsgType><![CDATA[%s]]></MsgType>
                        <Content><![CDATA[%s]]></Content>
                        <FuncFlag>0</FuncFlag>
                        </xml>";
            if($keyword == "?" || $keyword == "？")
            {
                $msgType = "text";
                $contentStr = date("Y-m-d H:i:s",time());
                $resultStr = sprintf($textTpl, $fromUsername, $toUsername, $time, $msgType, $contentStr);
                echo $resultStr;
            }
        }else{
            echo "";
            exit;
        }
    }
}
?>
```     

**第三步：依据接口文档实现业务逻辑**          
这里就在于自己开发了

