---
layout: post
title: Android实现电子邮箱客户端
date: 2016-4-18
categories: blog
tags: [android,java]
description: Android实现电子邮箱客户端
---

### 本文主要讲述了安卓平台上利用QQ邮箱SMTP协议，POP3协议发送与接收消息的实现   

### 准备过程 

1. 下载jar包
2.  在QQ邮箱中手动开启SMTP，POP3

### 参考链接 

[QQ邮箱开启SMTP服务的步骤_百度经验](http://jingyan.baidu.com/article/0f5fb099dffe7c6d8334ea31.html)

### 实现 

### 登陆与验证  

### 登陆界面，主要是两个输入框

```
package com.email;
import com.email.app.MyApplication;
import com.email.utils.EmailFormatUtil;
import com.email.utils.HttpUtil;

import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.text.Editable;
import android.text.TextUtils;
import android.text.TextWatcher;
import android.view.View;
import android.view.Window;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.CompoundButton;
import android.widget.CompoundButton.OnCheckedChangeListener;
import android.widget.EditText;
import android.widget.Toast;
import android.app.Activity;
import android.app.ProgressDialog;
import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
public class LoginActivity extends Activity implements TextWatcher, OnClickListener{
    private EditText emailAddress;
    private EditText password;
    private Button clearAddress;
    private Button emailLogin;
    private ProgressDialog dialog;
    private SharedPreferences sp;
    private CheckBox cb_remenber;
    private CheckBox cb_autologin;
    private Handler handler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            if(MyApplication.session==null){
                dialog.dismiss();
                Toast.makeText(LoginActivity.this, "账号或密码错误", Toast.LENGTH_SHORT).show();
            }else{
                dialog.dismiss();
                Intent intent=new Intent(LoginActivity.this, HomeActivity.class);
                startActivity(intent);
                finish();
                //Toast.makeText(LoginActivity.this, "登入成功", Toast.LENGTH_SHORT).show();
            }
            super.handleMessage(msg);
        }
        
    };
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.email_login);
        sp=getSharedPreferences("config", Context.MODE_APPEND);
        initView();
        isRemenberPwd();
    }
    
    /**
     * 初始化数据
     */
    private void initView(){
        emailAddress=(EditText) findViewById(R.id.emailAddress);
        password=(EditText) findViewById(R.id.password);
        clearAddress=(Button) findViewById(R.id.clear_address);
        emailLogin=(Button) findViewById(R.id.login_btn);
        cb_remenber=(CheckBox) findViewById(R.id.remenberPassword);
        cb_autologin=(CheckBox) findViewById(R.id.autoLogin);
        
        clearAddress.setOnClickListener(this);
        emailAddress.addTextChangedListener(this);
        emailLogin.setOnClickListener(this);
        
        cb_remenber.setOnClickListener(this);
        cb_autologin.setOnClickListener(this);
        
    }
    
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
        case R.id.clear_address:
            emailAddress.setText("");
            break;
        case R.id.login_btn:
            loginEmail();
            break;
        case R.id.remenberPassword:
            remenberPwd();
            break;
        case R.id.autoLogin:
            break;
        }
    }
    
    /**
     * 是否记住密码
     */
    private void isRemenberPwd(){
        boolean isRbPwd=sp.getBoolean("isRbPwd", false);
        if(isRbPwd){
            String addr=sp.getString("address", "");
            String pwd=sp.getString("password", "");
            emailAddress.setText(addr);
            password.setText(pwd);
            cb_remenber.setChecked(true);
        }
    }
    
    /**
     * 记住密码
     */
    private void remenberPwd(){
        boolean isRbPwd=sp.getBoolean("isRbPwd", false);
        if(isRbPwd){
            sp.edit().putBoolean("isRbPwd", false).commit();
            cb_remenber.setChecked(false);
        }else{
            sp.edit().putBoolean("isRbPwd", true).commit();
            sp.edit().putString("address", emailAddress.getText().toString().trim()).commit();
            sp.edit().putString("password", password.getText().toString().trim()).commit();
            cb_remenber.setChecked(true);
            
        }
    }
    
    /**
     * 登入邮箱
     */
    private void loginEmail(){
        String address=emailAddress.getText().toString().trim();
        String pwd=password.getText().toString().trim();
        if(TextUtils.isEmpty(address)){
            Toast.makeText(LoginActivity.this, "地址不能为空", Toast.LENGTH_SHORT).show();
            return;
        }else{
            if(TextUtils.isEmpty(pwd)){
                Toast.makeText(LoginActivity.this, "密码不能为空", Toast.LENGTH_SHORT).show();
                return;
            }
        }
        /**
         * 校验邮箱格式
         */
        if(!EmailFormatUtil.emailFormat(address)){
            Toast.makeText(LoginActivity.this, "邮箱格式不正确", Toast.LENGTH_SHORT).show();
        }else{
            String host="smtp."+address.substring(address.lastIndexOf("@")+1);
            MyApplication.info.setMailServerHost(host);
            MyApplication.info.setMailServerPort("465");
            MyApplication.info.setUserName(address);
            MyApplication.info.setPassword(pwd);
            MyApplication.info.setValidate(true);
            
            /**
             * 进度条
             */
            dialog=new ProgressDialog(LoginActivity.this);
            dialog.setMessage("正在登入，请稍后");
            dialog.show();
            
            /**
             * 访问网络
             */
            new Thread(){
                @Override
                public void run() {     
                    //登入操作
                    HttpUtil util=new HttpUtil();
                    MyApplication.session=util.login();
                    Message message=handler.obtainMessage();
                    message.sendToTarget();
                }
                
            }.start();
        }
    }
    
     
        /**
         * 文本监听事件
         * 
         */
        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {
                if(!TextUtils.isEmpty(s)){
                    clearAddress.setVisibility(View.VISIBLE);
                }else{
                    clearAddress.setVisibility(View.INVISIBLE);
                }
            
        }
    
    @Override
    public void afterTextChanged(Editable s) {

    }

    @Override
    public void beforeTextChanged(CharSequence s, int start, int count,
            int after) {
        
    }
    

}
```  


### 点击登陆会跳转到此处，HttpUtils如下 ，主要包括验证是否成功登陆与发送消息的方法，
其中创建了一个密码验证器，并且根据邮件会话属性和密码验证器构造一个发送邮件的session


```
package com.email.utils;
import java.io.UnsupportedEncodingException;
import java.util.Date;
import java.util.List;

import javax.activation.CommandMap;
import javax.activation.DataHandler;
import javax.activation.FileDataSource;
import javax.activation.MailcapCommandMap;
import javax.mail.Address;
import javax.mail.BodyPart;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Multipart;
import javax.mail.Session;
import javax.mail.Store;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;
import javax.mail.internet.MimeUtility;
import com.email.app.MyApplication;
import com.email.bean.Attachment;
import com.email.bean.MailInfo;
import com.email.bean.MyAuthenticator;
public class HttpUtil {
    /**
     * 连接邮箱
     * @param info
     * @return
     */
    public Session login(){
        //连接服务器
        Session session=isLoginRight(MyApplication.info);
        return session;
    }
    
    /**
     * 登入操作
     * @param info
     * @return
     */
    private Session isLoginRight(MailInfo info) {
        //判断是否要登入验证
        MyAuthenticator authenticator=null;
        if(info.isValidate()){
            //创建一个密码验证器
            authenticator=new MyAuthenticator(info.getUserName(), info.getPassword());
        }
        // 根据邮件会话属性和密码验证器构造一个发送邮件的session
        Session sendMailSession=Session.getDefaultInstance(info.getProperties(), authenticator);
        try {
            Transport transport=sendMailSession.getTransport("smtp");
            transport.connect(info.getMailServerHost(), info.getUserName(), info.getPassword());
        } catch (MessagingException e) {
            e.printStackTrace();
            return null;
        }
        return sendMailSession;
    }
    
    /**
     * 以文本格式发送邮件
     * 
     * @param <T>
     * 
     * @param mailInfo
     *            待发送的邮件的信息
     */
    public boolean sendTextMail(MailInfo mailInfo, Session sendMailSession) {
        // 判断是否需要身份认证
        try {
            // 根据session创建一个邮件消息
            Message mailMessage = new MimeMessage(sendMailSession);
            // 创建邮件发送者地址
            Address address=new InternetAddress(mailInfo.getFromAddress());
            // 设置邮件消息的发送者
            mailMessage.setFrom(address);
            // 创建邮件的接收者地址，并设置到邮件消息中
            Address[] tos = null;
            String[] receivers = mailInfo.getReceivers();
            if (receivers != null) {
                // 为每个邮件接收者创建一个地址
                tos = new InternetAddress[receivers.length];
                for (int i = 0; i < receivers.length; i++) {
                    tos[i] = new InternetAddress(receivers[i]);
                }
            } else {
                return false;
            }
            // Message.RecipientType.TO属性表示接收者的类型为TO
            mailMessage.setRecipients(Message.RecipientType.TO, tos);
            // 设置邮件消息的主题
            mailMessage.setSubject(mailInfo.getSubject());
            // 设置邮件消息发送的时间
            mailMessage.setSentDate(new Date());
            // 设置邮件消息的主要内容
            String mailContent = mailInfo.getContent();

            Multipart mm = new MimeMultipart();// 新建一个MimeMultipart对象用来存放多个BodyPart对象
            // 设置信件文本内容
            BodyPart mdp = new MimeBodyPart();// 新建一个存放信件内容的BodyPart对象
            mdp.setContent(mailContent, "text/html;charset=gb2312");// 给BodyPart对象设置内容和格式/编码方式
            mm.addBodyPart(mdp);// 将含有信件内容的BodyPart加入到MimeMultipart对象中

            Attachment affInfos;
            FileDataSource fds1;
            List<Attachment> list = mailInfo.getAttachmentInfos();
                for (int i = 0; i < list.size(); i++) {
                    affInfos = list.get(i);
                    fds1 = new FileDataSource(affInfos.getFilePath());
                    mdp = new MimeBodyPart();
                    mdp.setDataHandler(new DataHandler(fds1));
                    try {
                        mdp.setFileName(MimeUtility.encodeText(fds1.getName()));
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                    }
                    mm.addBodyPart(mdp);
                }
            mailMessage.setContent(mm);
            mailMessage.saveChanges();
            
            
            // 设置邮件支持多种格式
            MailcapCommandMap mc = (MailcapCommandMap) CommandMap.getDefaultCommandMap();
            mc.addMailcap("text/html;; x-java-content-handler=com.sun.mail.handlers.text_html");
            mc.addMailcap("text/xml;; x-java-content-handler=com.sun.mail.handlers.text_xml");
            mc.addMailcap("text/plain;; x-java-content-handler=com.sun.mail.handlers.text_plain");
            mc.addMailcap("multipart/*;; x-java-content-handler=com.sun.mail.handlers.multipart_mixed");
            mc.addMailcap("message/rfc822;; x-java-content-handler=com.sun.mail.handlers.message_rfc822");
            CommandMap.setDefaultCommandMap(mc);
            
            // 发送邮件
            Transport.send(mailMessage);
            return true;
        } catch (MessagingException ex) {
            ex.printStackTrace();
        }
        return false;
    }
}
```     

#### HttpUtils中用到了MailInfo中一些属性，MailInfo是邮件基本信息javabean,其中配置了邮件会话属性，和其他一些属性，内容如下

由于现在是SSL方法验证，似乎都要加上
```
p.put("mail.smtp.starttls.enable", "true");
        //需要加上
        p.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
        p.put("mail.smtp.socketFactory.port", "465");
```

刚开始没加就出错了


```
package com.email.bean;

import java.io.Serializable;
import java.util.List;
import java.util.Properties;

/**
 * 邮件的基本信息
 * @author Administrator
 *
 */
public class MailInfo implements Serializable {
    private static final long serialVersionUID = 1L;
    // 发送邮件的服务器的IP和端口
    private String mailServerHost;
    private String mailServerPort = "465";
    // 登陆邮件发送服务器的用户名和密码
    private String userName;
    private String password;
    // 是否需要身份验证
    private boolean validate = false;
    // 邮件发送者的地址
    private String fromAddress;
    // 邮件主题
    private String subject;
    // 邮件的文本内容
    private String content;
    // 邮件附件的路径
    private List<Attachment> attachmentInfos;
    // 邮件的接收者，可以有多个
    private String[] receivers;

    /**
     * 获得邮件会话属性
     */
    public Properties getProperties() {
        Properties p = new Properties();
        p.put("mail.smtp.host", this.mailServerHost);
        p.put("mail.smtp.port", this.mailServerPort);
        p.put("mail.smtp.auth", validate ? "true" : "false");
        
        p.put("mail.smtp.starttls.enable", "true");
        //需要加上
        p.put("mail.smtp.socketFactory.class", "javax.net.ssl.SSLSocketFactory");
        p.put("mail.smtp.socketFactory.port", "465");
        
        

        
        return p;
    }

    public String[] getReceivers() {
        return receivers;
    }

    public void setReceivers(String[] receivers) {
        this.receivers = receivers;
    }

    public String getMailServerHost() {
        return mailServerHost;
    }

    public void setMailServerHost(String mailServerHost) {
        this.mailServerHost = mailServerHost;
    }

    public String getMailServerPort() {
        return mailServerPort;
    }

    public void setMailServerPort(String mailServerPort) {
        this.mailServerPort = mailServerPort;
    }

    public boolean isValidate() {
        return validate;
    }

    public void setValidate(boolean validate) {
        this.validate = validate;
    }


    public List<Attachment> getAttachmentInfos() {
        return attachmentInfos;
    }

    public void setAttachmentInfos(List<Attachment> attachmentInfos) {
        this.attachmentInfos = attachmentInfos;
    }

    

    public String getFromAddress() {
        return fromAddress;
    }

    public void setFromAddress(String fromAddress) {
        this.fromAddress = fromAddress;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getSubject() {
        return subject;
    }

    public void setSubject(String subject) {
        this.subject = subject;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String textContent) {
        this.content = textContent;
    }
}


```  


### session是Application中声明的静态全局变量，info也是

```
package com.email.app;
import java.io.InputStream;
import java.util.ArrayList;

import javax.mail.Session;
import javax.mail.Store;

import com.email.bean.MailInfo;

import android.app.Application;

public class MyApplication extends Application {
    public static Session session = null;
    public static Store getStore() {
        return store;
    }

    public static void setStore(Store store) {
        MyApplication.store = store;
    }

    public static MailInfo info=new MailInfo();
    
    private static Store store;
    private ArrayList<InputStream> attachmentsInputStreams;

    public ArrayList<InputStream> getAttachmentsInputStreams() {
        return attachmentsInputStreams;
    }

    public void setAttachmentsInputStreams(ArrayList<InputStream> attachmentsInputStreams) {
        this.attachmentsInputStreams = attachmentsInputStreams;
    }
    
}
```    

### 参考链接：
[JavaMail - qw765811529的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/qw765811529/article/details/50727785)


### 接收邮箱信息

待续  

### 效果如下

![这里写图片描述](http://img.blog.csdn.net/20160418210015201)

![这里写图片描述](http://img.blog.csdn.net/20160418210026857)

































