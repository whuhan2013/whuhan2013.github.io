---
layout: post
title: android之Token保持登录状态
date: 2016-7-21
categories: blog
tags: [android]
description: android
---


**Token是什么？**

Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。

**Token的引入**

Token是在客户端频繁向服务端请求数据，服务端频繁的去数据库查询用户名和密码并进行对比，判断用户名和密码正确与否，并作出相应提示，在这样的背景下，Token便应运而生


**使用Token的目的**

Token的目的是为了减轻服务器的压力，减少频繁的查询数据库，使服务器更加健壮,当然也有避免密码明文传输，安全的作用。

**参考链接:**[Android客户端与服务器交互中的token - ZhangGeng's Blog - 博客频道 - CSDN.NET](http://blog.csdn.net/watermusicyes/article/details/47016113)

**如何使用TOKEN** 

客户端：客户端只需携带用户名和密码登陆即可。

服务器端：客户端接收到用户名和密码后并判断，如果正确了就将本地编码生成token，并利用Memcached缓存在服务器中，然后token返回客户端，以后请求只需要发送token就可以了。          

**参考链接：**[利用缓存实现APP端与服务器接口交互的Session控制 - 无心码农 - 博客园](http://www.cnblogs.com/liujiduo/p/5007043.html)


**生成token的算法** 

```
String key = UUID.randomUUID().toString().toUpperCase() +
        "|" + someImportantProjectToken +
        "|" + userName +
        "|" + creationDateTime;

StandardPBEStringEncryptor jasypt = new StandardPBEStringEncryptor();

...

// this is the authentication token user will send in order to use the web service
String authenticationToken = jasypt.encrypt(key);
```

**解密算法** 

```
import org.jasypt.encryption.pbe.StandardPBEStringEncryptor;
import org.jasypt.encryption.pbe.config.EnvironmentStringPBEConfig;

/**
 *把密文放到配置文件中的时候要注意：
 * ENC(密文)
 * @author 杨尚川
 */
public class ConfigEncryptUtils {
    public static void main(String[] args){
        //加密工具
        StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
        //加密配置
        EnvironmentStringPBEConfig config = new EnvironmentStringPBEConfig();
        config.setAlgorithm("PBEWithMD5AndDES");
        //自己在用的时候更改此密码
        config.setPassword("apdplat");
        //应用配置
        encryptor.setConfig(config);
        String ciphertext="azL9Cyp9H62r3eUgZ+TESw==";
        //解密
        String plaintext=encryptor.decrypt(ciphertext);
        System.out.println(ciphertext + " : " + plaintext);
    }
}
```

**参考链接** 

[security - Session management](https://stackoverflow.com/questions/4973454/session-management-how-to-generate-authentication-token-for-rest-service-je)

[如何在你的应用中使用Jasypt来保护你的数据库用户名和密码 - 杨尚川的个人页面 - 开源中国社区](http://my.oschina.net/apdplat/blog/405306)

