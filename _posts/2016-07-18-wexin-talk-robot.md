---
layout: post
title: 微信开发简单实例
date: 2016-7-18
categories: blog
tags: [微信开发]
description: 微信开发
---

**本文主要包括以下内容**          
1. 微信聊天机器人            


#### 微信聊天机器人               

利用图灵机器人接口实现微信聊天机器人     

```
<?php

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
            // if($keyword == "?" || $keyword == "？")
            // {
            //     $msgType = "text";
            //     $contentStr = date("Y-m-d H:i:s",time());
            //     $resultStr = sprintf($textTpl, $fromUsername, $toUsername, $time, $msgType, $contentStr);
            //     echo $resultStr;
            // }
            $msgType="text";
            $resulttalk=$this->tuling($keyword);
            $resultStr = sprintf($textTpl, $fromUsername, $toUsername, $time, $msgType, $resulttalk);
            echo $resultStr;
        }else{
            echo "";
            exit;
        }
    }

        // 图灵机器人
    function tuling($keyword) {
        $key="82afb9ed426e8506be0fef7f70ed3f16";//api key到这里申请
        $api_url = "http://www.tuling123.com/openapi/api?key=".$key."&info=". $keyword;
        
        $result = file_get_contents ( $api_url );
        $result = json_decode ( $result, true );
        
        switch ($result ['code']) {
            case '100000':
                $resulttext=$result['text'];
                return $resulttext;
                break;
          case '200000' :
            $text = $result ['text'] . ',<a href="' . $result ['url'] . '">点击进入</a>';
            return $text;
            break;
          case '301000' :
        $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => $result['list'][$i]['author'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '302000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['article'],
                    'Description' => $result['list'][$i]['source'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '304000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i< $length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => $result['list'][$i]['count'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '305000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['start'] . '--' . $result['list'][$i]['terminal'],
                    'Description' => $result['list'][$i]['starttime'] . '--' . $result['list'][$i]['endtime'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '306000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['flight'] . '--' . $result['list'][$i]['route'],
                    'Description' => $result['list'][$i]['starttime'] . '--' . $result['list'][$i]['endtime'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '307000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => $result['list'][$i]['info'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '308000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => $result['list'][$i]['info'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '309000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => '价格 : ' . $result['list'][$i]['price'] . ' 满意度 : ' . $result['list']['satisfaction'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '310000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['number'],
                    'Description' => $result['list'][$i]['info'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '311000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => '价格 : ' . $result['list'][$i]['price'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          case '312000' :
            $length = count($result['list']) > 9 ? 9 :count($result['list']);
            for($i= 0;$i<$length;$i++){
                $articles [$i] = array (
                    'Title' => $result['list'][$i]['name'],
                    'Description' => '价格 : ' . $result['list'][$i]['price'],
                    'PicUrl' => $result['list'][$i]['icon'],
                    'Url' => $result['list'][$i]['detailurl']
                );
            }
            return $articles;
            break;
          default :
            if (emptyempty ( $result ['text'] )) {
                return false;
            } else {
                return $result ['text'] ;
            }
        }
        
    } 


}
?>
```

![](https://raw.githubusercontent.com/whuhan2013/ImageRepertory/master/php/p1.jpg)


 