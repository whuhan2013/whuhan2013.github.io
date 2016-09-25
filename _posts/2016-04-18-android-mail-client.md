---
layout: post
title: Android实现电子邮箱客户端
date: 2016-4-18
categories: blog
tags: [android,java]
description: Android实现电子邮箱客户端
---

### 本文主要讲述了安卓平台上利用QQ邮箱SMTP协议，POP3协议发送与接收消息的实现   

### 发送邮件核心代码  

```
 import java.security.Security;    
  import java.util.Date;    
  import java.util.Properties;    
        
   import javax.mail.Authenticator;    
    import javax.mail.Message;    
    import javax.mail.MessagingException;    
   import javax.mail.PasswordAuthentication;    
   import javax.mail.Session;    
   import javax.mail.Transport;    
    import javax.mail.internet.AddressException;    
    import javax.mail.internet.InternetAddress;    
   import javax.mail.internet.MimeMessage;    

    /**    
     * 使用Gmail发送邮件    
     * @author Winter Lau    
     */    
    public class GmailSender {    
         
     public static void main(String[] args) throws AddressException, MessagingException {    
      Security.addProvider(new com.sun.net.ssl.internal.ssl.Provider());    
      final String SSL_FACTORY = "javax.net.ssl.SSLSocketFactory";    
      // Get a Properties object    
      Properties props = System.getProperties();    
      props.setProperty("mail.smtp.host", "smtp.qq.com");    
      props.setProperty("mail.smtp.socketFactory.class", SSL_FACTORY);    
      props.setProperty("mail.smtp.socketFactory.fallback", "false");    
     props.setProperty("mail.smtp.port", "465");    
      props.setProperty("mail.smtp.socketFactory.port", "465");    
      props.put("mail.smtp.auth", "true");    
      final String username = "yourusername";    
      final String password = "yourpassword";    
     Session session = Session.getDefaultInstance(props, new Authenticator(){    
          protected PasswordAuthentication getPasswordAuthentication() {    
              return new PasswordAuthentication(username, password);    
          }});    
         
           // -- Create a new message --    
     Message msg = new MimeMessage(session);    
        
     // -- Set the FROM and TO fields --    
     msg.setFrom(new InternetAddress(username + "@qq.com"));    
      msg.setRecipients(Message.RecipientType.TO,    
       InternetAddress.parse("yourusername@qq.com",false));    
      msg.setSubject("Hello");    
      msg.setText("How are you");    
     msg.setSentDate(new Date());    
      Transport.send(msg);    
           
      System.out.println("Message sent.");    
     }    
    }
```      

### 接收邮件核心代码  

```
 import java.io.UnsupportedEncodingException;    
    import java.security.*;    
    import java.util.Properties;    
    import javax.mail.*;    
    import javax.mail.internet.InternetAddress;    
    import javax.mail.internet.MimeUtility;    
         
   /**    
    * 用于收取Gmail邮件    
     * @author Winter Lau    
    */    
   public class GmailFetch {    
         
    public static void main(String argv[]) throws Exception {    
        
      Security.addProvider(new com.sun.net.ssl.internal.ssl.Provider());    
     final String SSL_FACTORY = "javax.net.ssl.SSLSocketFactory";    
        
      // Get a Properties object    
      Properties props = System.getProperties();    
      props.setProperty("mail.pop3.socketFactory.class", SSL_FACTORY);    
     props.setProperty("mail.pop3.socketFactory.fallback", "false");    
      props.setProperty("mail.pop3.port", "995");    
      props.setProperty("mail.pop3.socketFactory.port", "995");    
         
      //以下步骤跟一般的JavaMail操作相同    
      Session session = Session.getDefaultInstance(props,null);    
         
     //请将红色部分对应替换成你的邮箱帐号和密码    
     URLName urln = new URLName("pop3","pop.qq.com",995,null,    
       "yourusername@qq.com", "yourpassword");    
    Store store = session.getStore(urln);    
      Folder inbox = null;    
      try {    
       store.connect();    
       inbox = store.getFolder("INBOX");    
       inbox.open(Folder.READ_ONLY);    
       FetchProfile profile = new FetchProfile();    
       profile.add(FetchProfile.Item.ENVELOPE);    
       Message[] messages = inbox.getMessages();    
      inbox.fetch(messages, profile);    
       System.out.println("收件箱的邮件数：" + messages.length);    
       for (int i = 0; i < messages.length; i++) {    
       //邮件发送者    
       String from = decodeText(messages[i].getFrom()[0].toString());    
        InternetAddress ia = new InternetAddress(from);    
       System.out.println("FROM:" + ia.getPersonal()+'('+ia.getAddress()+')');    
        //邮件标题    
        System.out.println("TITLE:" + messages[i].getSubject());    
        //邮件大小    
        System.out.println("SIZE:" + messages[i].getSize());    
        //邮件发送时间    
        System.out.println("DATE:" + messages[i].getSentDate());    
       }    
      } finally {    
       try {    
        inbox.close(false);    
      } catch (Exception e) {}    
      try {    
        store.close();    
       } catch (Exception e) {}    
      }    
     }    
         
    protected static String decodeText(String text)    
       throws UnsupportedEncodingException {    
      if (text == null)    
       return null;    
     if (text.startsWith("=?GB") || text.startsWith("=?gb"))    
      text = MimeUtility.decodeText(text);    
     else    
      text = new String(text.getBytes("ISO8859_1"));    
     return text;    
    }    
         
   }    
```  

### 注意，接收邮件使用java客户端时可以正常得到数据，在使用安卓的时候，就会卡在store.connect(),原因不详，无法解决

### 问题已解决，安卓似乎强行并不允许连接网络的操作在主线程中，将store.connnect()放到子线程中即可，不然会卡住

### 注意，发邮件时，要导入activation.jar,不然的话会报java.lang.noClassDefFoundError:javax.activation.DataHandler 


### 最后完成版本   

### 主界面

```
package com.zj.myemail;



import android.annotation.SuppressLint;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.AlertDialog.Builder;
import android.content.ContentValues;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.DialogInterface.OnClickListener;
import android.net.Uri;
import android.os.Bundle;
import android.view.KeyEvent;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.view.Window;
import android.widget.BaseExpandableListAdapter;
import android.widget.EditText;
import android.widget.ExpandableListView;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;
import android.widget.ExpandableListView.OnChildClickListener;
import android.widget.ExpandableListView.OnGroupClickListener;

public class MainActivity extends Activity {

    private ExpandableListView expendView;
    private int []group_click=new int[5];
    private long mExitTime=0;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);
         final MyExpendAdapter adapter=new MyExpendAdapter();
        
        expendView=(ExpandableListView) findViewById(R.id.list);
        expendView.setGroupIndicator(null);  //设置默认图标不显示
        expendView.setAdapter(adapter);
        
        //一级点击事件
        expendView.setOnGroupClickListener(new OnGroupClickListener() {
            @Override
            public boolean onGroupClick(ExpandableListView parent, View v,
                    int groupPosition, long id) {
            
                group_click[groupPosition]+=1;
                adapter.notifyDataSetChanged();
                return false;
            }
        });
        
        //二级点击事件
        expendView.setOnChildClickListener(new OnChildClickListener() { 
            @Override
            public boolean onChildClick(ExpandableListView parent, View v,
                    int groupPosition, int childPosition, long id) {
                //可在这里做点击事件
                if(groupPosition==0&&childPosition==1){
                    /*AlertDialog.Builder builder=new Builder(MainActivity.this);
                    builder.setTitle("添加联系人");
                    
                    View view=getLayoutInflater().inflate(R.layout.email_add_address, null);
                    final EditText name=(EditText) view.findViewById(R.id.name);
                    final EditText addr=(EditText) view.findViewById(R.id.address);

                    builder.setView(view);
                    builder.setPositiveButton("确定", new OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            insertAddress(name.getText().toString().trim(),addr.getText().toString().trim());
                        }
                    });
                    builder.setNegativeButton("取消", null);
                    builder.show();*/
                }else if(groupPosition==0&&childPosition==0){
                    //Intent intent=new Intent(MainActivity.this, MailConstactsActivity.class);
                    //startActivity(intent);
                }else if(groupPosition==1&&childPosition==0){
                    Intent intent=new Intent(MainActivity.this, MailEditActivity.class);
                    startActivity(intent);
                }else if(groupPosition==1&&childPosition==1){
                    //Intent intent=new Intent(MainActivity.this, MailCaogaoxiangActivity.class);
                    //startActivity(intent);
                }else if(groupPosition==2&&childPosition==0){
                    Intent intent=new Intent(MainActivity.this, MailBoxActivity.class);
                    intent.putExtra("TYPE", "INBOX");
                    intent.putExtra("status", 0);//全部
                    startActivity(intent);
                }else if(groupPosition==2&&childPosition==1){
                    //Intent intent=new Intent(MainActivity.this, MailBoxActivity.class);
                    //intent.putExtra("TYPE", "INBOX");
                    //intent.putExtra("status", 1);//未读
                    //startActivity(intent);
                }else if(groupPosition==2&&childPosition==2){
                    //Intent intent=new Intent(MainActivity.this, MailBoxActivity.class);
                    //intent.putExtra("TYPE", "INBOX");
                    //intent.putExtra("status", 2);//已读
                    //startActivity(intent);
                }
                adapter.notifyDataSetChanged();
                return false;
            }
        });
        
    }
    
    /**
     * 添加联系人
     */
    /*private void insertAddress(String user,String address){
        if(user==null){
            Toast.makeText(HomeActivity.this, "用户名不能为空", Toast.LENGTH_SHORT).show();
        }else{
            if(!EmailFormatUtil.emailFormat(address)){
                Toast.makeText(HomeActivity.this, "邮箱格式不正确", Toast.LENGTH_SHORT).show();
            }else{
                Uri uri=Uri.parse("content://com.emailconstantprovider");
                ContentValues values=new ContentValues();
                values.put("mailfrom", MyApplication.info.getUserName());
                values.put("name", user);
                values.put("address", address);
                getContentResolver().insert(uri, values);
                
                Toast.makeText(HomeActivity.this, "添加数据成功", Toast.LENGTH_SHORT).show();
            }
        }
        
        
    }*/
    
    /**
     * 适配器
     * @author Administrator
     *
     */
    private class MyExpendAdapter extends BaseExpandableListAdapter{
        
        /**
         * pic state
         */
        //int []group_state=new int[]{R.drawable.group_right,R.drawable.group_down};
        
        /**
         * group title
         */
        String []group_title=new String[]{"联系人","写邮件","收件箱"};
        
        /**
         * child text
         */
        String [][] child_text=new String [][]{
                {"联系人列表","添加联系人"},
                {"新邮件","草稿箱"},
                {"全部邮件","未读邮件","已读邮件"},};
        int [][] child_icons=new int[][]{
                {R.drawable.listlianxiren,R.drawable.tianjia},
                {R.drawable.xieyoujian,R.drawable.caogaoxiang},
                {R.drawable.all,R.drawable.notread,R.drawable.hasread},
        };
        
        /**
         * 获取一级标签中二级标签的内容
         */
        @Override
        public Object getChild(int groupPosition, int childPosition) {
            return child_text[groupPosition][childPosition];
        }
        
        /**
         * 获取二级标签ID
         */
        @Override
        public long getChildId(int groupPosition, int childPosition) {
            return childPosition;
        }
        /**
         * 对一级标签下的二级标签进行设置
         */
        @SuppressLint("SimpleDateFormat")
        @Override
        public View getChildView(int groupPosition, int childPosition,
                boolean isLastChild, View convertView, ViewGroup parent) {
            convertView=getLayoutInflater().inflate(R.layout.email_child, null);
            TextView tv=(TextView) convertView.findViewById(R.id.tv);
            tv.setText(child_text[groupPosition][childPosition]);
            
            ImageView iv=(ImageView) convertView.findViewById(R.id.child_icon);
            iv.setImageResource(child_icons[groupPosition][childPosition]);
            return convertView;
        }
        
        /**
         * 一级标签下二级标签的数量
         */
        @Override
        public int getChildrenCount(int groupPosition) {
            return child_text[groupPosition].length;
        }
        
        /**
         * 获取一级标签内容
         */
        @Override
        public Object getGroup(int groupPosition) {
            return group_title[groupPosition];
        }
        
        /**
         * 一级标签总数
         */
        @Override
        public int getGroupCount() {
            return group_title.length;
        }
        
        /**
         * 一级标签ID
         */
        @Override
        public long getGroupId(int groupPosition) {
            return groupPosition;
        }
        /**
         * 对一级标签进行设置
         */
        @Override
        public View getGroupView(int groupPosition, boolean isExpanded,
                View convertView, ViewGroup parent) {
            convertView=getLayoutInflater().inflate(R.layout.email_group, null);
            
            ImageView icon=(ImageView) convertView.findViewById(R.id.icon);
            ImageView iv=(ImageView) convertView.findViewById(R.id.iv);
            TextView tv=(TextView) convertView.findViewById(R.id.iv_title);
            
            iv.setImageResource(R.drawable.group_right);
            tv.setText(group_title[groupPosition]);
            
            if(groupPosition==0){
                icon.setImageResource(R.drawable.constants);
            }else if(groupPosition==1){
                icon.setImageResource(R.drawable.mailto);
            }else if(groupPosition==2){
                icon.setImageResource(R.drawable.mailbox);
            }
            
            if(group_click[groupPosition]%2==0){
                iv.setImageResource(R.drawable.group_right);
            }else{
                iv.setImageResource(R.drawable.group_down);
            }
            
            
            return convertView;
        }
        /**
         * 指定位置相应的组视图
         */
        @Override
        public boolean hasStableIds() {
            return true;
        }
        
        /**
         *  当选择子节点的时候，调用该方法
         */
        @Override
        public boolean isChildSelectable(int groupPosition, int childPosition) {
            return true;
        }
        
    }
    
    /**
     * 返回退出系统
     */
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if(keyCode==KeyEvent.KEYCODE_BACK){
            if((System.currentTimeMillis()-mExitTime)<2000){
                android.os.Process.killProcess(android.os.Process.myPid());
            }else{
                Toast.makeText(MainActivity.this, "再按一次退出程序", Toast.LENGTH_SHORT).show();
                mExitTime=System.currentTimeMillis();
            }
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
}
```   


### 收件箱  

```

package com.zj.myemail;

import android.app.Activity;
import android.app.ProgressDialog;
import android.content.ContentValues;
import android.content.Context;
import android.content.Intent;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.os.Handler;

import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.ViewGroup;
import android.view.Window;
import android.widget.BaseAdapter;
import android.widget.DialerFilter;
import android.widget.ListView;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;

import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.lang.ref.WeakReference;
import java.security.Security;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

import javax.mail.FetchProfile;
import javax.mail.Folder;
import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Store;
import javax.mail.URLName;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeUtility;



public class MailBoxActivity extends Activity {

    private ArrayList<Email> mailslist = new ArrayList<Email>();
    private ArrayList<ArrayList<InputStream>> attachmentsInputStreamsList = new ArrayList<ArrayList<InputStream>>();
    private String type;
    private int status;
    private MyAdapter myAdapter;
    private ListView lv_box;
    //private List<MailReceiver> mailReceivers;
    private ProgressDialog dialog;
  
    private List<String> messageids;
    private Handler handler=new Handler(){
        public void handleMessage(android.os.Message msg) {
            dialog.dismiss();
            
             myAdapter = new MyAdapter();
             lv_box.setAdapter(myAdapter);
        };
        
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        super.onCreate(savedInstanceState);
       
        
        setContentView(R.layout.email_mailbox);
        initView();

    }

    private void initView() {
        lv_box = (ListView) findViewById(R.id.lv_box);
       
        
        dialog=new ProgressDialog(this);
        dialog.setMessage("正加载");
        dialog.show();
        
        
        Security.addProvider(new com.sun.net.ssl.internal.ssl.Provider());    
         final String SSL_FACTORY = "javax.net.ssl.SSLSocketFactory";    
            
          // Get a Properties object    
          Properties props = System.getProperties();    
          props.setProperty("mail.pop3.socketFactory.class", SSL_FACTORY);    
         props.setProperty("mail.pop3.socketFactory.fallback", "false");    
          props.setProperty("mail.pop3.port", "995");    
          props.setProperty("mail.pop3.socketFactory.port", "995");    
             
          //以下步骤跟一般的JavaMail操作相同    
          final Session session = Session.getDefaultInstance(props,null);    
             
         //请将红色部分对应替换成你的邮箱帐号和密码    
         final URLName urln = new URLName("pop3","pop.qq.com",995,null,    
           "yourusername@qq.com", "yourpassword");   
        
         
         
         new Thread(new Runnable() {
            
            @Override
            public void run() {
                // TODO Auto-generated method stub
                 Store store=null;
                  Folder inbox = null; 
                try { 
                    
                     store = session.getStore(urln);    
                        
                     Log.i("test","here1");
                       store.connect();    
                       Log.i("test","here2");
                       inbox = store.getFolder("INBOX");    
                       inbox.open(Folder.READ_ONLY);    
                       FetchProfile profile = new FetchProfile();    
                       profile.add(FetchProfile.Item.ENVELOPE);    
                       Message[] messages = inbox.getMessages();    
                      inbox.fetch(messages, profile);  
                      Log.i("test",messages.length+"");
                       System.out.println("收件箱的邮件数：" + messages.length);    
                       for (int i = 0; i < messages.length; i++) {    
                       //邮件发送者    
                       String from = decodeText(messages[i].getFrom()[0].toString());    
                        InternetAddress ia = new InternetAddress(from);    
                       System.out.println("FROM:" + ia.getPersonal()+'('+ia.getAddress()+')');    
                        //邮件标题    
                        System.out.println("TITLE:" + messages[i].getSubject());    
                        //邮件大小    
                        System.out.println("SIZE:" + messages[i].getSize());    
                        //邮件发送时间    
                        System.out.println("DATE:" + messages[i].getSentDate());   
                        Email email=new Email();
                        email.setFrom(ia.getPersonal()+'('+ia.getAddress()+')');
                        email.setSubject( messages[i].getSubject());
                        email.setSentdata(messages[i].getSentDate()+"");
                        
                       mailslist.add(email);
                        
                       }    
                       
                       handler.sendEmptyMessage(0);
                      } catch(Exception e)
                      {
                          
                      }
                      finally {    
                       try {    
                        inbox.close(false);    
                      } catch (Exception e) {}    
                      try {    
                        store.close();    
                       } catch (Exception e) {}    
                      }  
            }
        }).start();
        
    }
    
    protected static String decodeText(String text)    
               throws UnsupportedEncodingException {    
              if (text == null)    
               return null;    
             if (text.startsWith("=?GB") || text.startsWith("=?gb"))    
              text = MimeUtility.decodeText(text);    
             else    
              text = new String(text.getBytes("ISO8859_1"));    
             return text;    
            }    
    
   
    
    /**
     * 适配器
     * @author Administrator
     *
     */
    private class MyAdapter extends BaseAdapter {
        @Override
        public int getCount() {
            return mailslist.size();
        }

        @Override
        public Object getItem(int position) {
            return mailslist.get(position);
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(final int position, View convertView, ViewGroup parent) {
            convertView = LayoutInflater.from(MailBoxActivity.this).inflate(R.layout.email_mailbox_item, null);
            
            TextView tv_from = (TextView) convertView.findViewById(R.id.tv_from);
            tv_from.setText(mailslist.get(position).getFrom());
            
            TextView tv_sentdate = (TextView) convertView.findViewById(R.id.tv_sentdate);
            tv_sentdate.setText(mailslist.get(position).getSentdata());
           
            TextView tv_subject = (TextView) convertView.findViewById(R.id.tv_subject);
            tv_subject.setText(mailslist.get(position).getSubject());
            
            return convertView;
        }

    }
    
  
   
}
```    

### 发送邮件  


```
package com.zj.myemail;

import java.security.Security;
import java.util.Date;
import java.util.Properties;

import javax.mail.Authenticator;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

import android.app.Activity;
import android.app.ProgressDialog;
import android.os.Bundle;
import android.os.Handler;
import android.view.KeyEvent;
import android.view.View;
import android.view.Window;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.Toast;





public class MailEditActivity extends Activity {
    private EditText mail_to;
    private EditText mail_from;
    private EditText mail_topic;
    private EditText mail_content;

    private Button send;
    private ImageButton add_lianxiren;
    private ImageButton attachment;
    

    private int mailid = -1;

    private static final int SUCCESS = 1;
    private static final int FAILED = -1;
    private boolean isCaogaoxiang = true;
    private ProgressDialog dialog;

    Handler handler = new Handler() {

        public void handleMessage(android.os.Message msg) {
            dialog.dismiss();
            Toast.makeText(MailEditActivity.this, "发送成功", 0).show();
        };
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        super.onCreate(savedInstanceState);
        setContentView(R.layout.email_writer);
        init();
    }

    /**
     * 初始化
     */
    private void init() {
        mail_to = (EditText) findViewById(R.id.mail_to);
        mail_from = (EditText) findViewById(R.id.mail_from);
        mail_topic = (EditText) findViewById(R.id.mail_topic);
        mail_content = (EditText) findViewById(R.id.content);
        send = (Button) findViewById(R.id.send);
        attachment = (ImageButton) findViewById(R.id.add_att);
        add_lianxiren = (ImageButton) findViewById(R.id.add_lianxiren);

        mail_from.setText("yourusername@qq.com");
        mail_to.setText("yourusername@qq.com");

    }

    public void mySend(View view) {
        Toast.makeText(getApplicationContext(), "正在发送", 0).show();
        sendMail();
    }

    /**
     * 设置邮件数据
     */
    private void sendMail() {

        dialog = new ProgressDialog(this);
        dialog.setMessage("正在发送");
        dialog.show();
        new Thread(new Runnable() {

            @Override
            public void run() {
                // TODO Auto-generated method stub
                try {
                    Security.addProvider(new com.sun.net.ssl.internal.ssl.Provider());
                    final String SSL_FACTORY = "javax.net.ssl.SSLSocketFactory";
                    // Get a Properties object
                    Properties props = System.getProperties();
                    props.setProperty("mail.smtp.host", "smtp.qq.com");
                    props.setProperty("mail.smtp.socketFactory.class",
                            SSL_FACTORY);
                    props.setProperty("mail.smtp.socketFactory.fallback",
                            "false");
                    props.setProperty("mail.smtp.port", "465");
                    props.setProperty("mail.smtp.socketFactory.port", "465");
                    props.put("mail.smtp.auth", "true");
                    final String username = "yourusername";
                    final String password = "yourpassword";
                    Session session = Session.getDefaultInstance(props,
                            new Authenticator() {
                                protected PasswordAuthentication getPasswordAuthentication() {
                                    return new PasswordAuthentication(username,
                                            password);
                                }
                            });

                    // -- Create a new message --
                    final Message msg = new MimeMessage(session);

                    // -- Set the FROM and TO fields --
                    msg.setFrom(new InternetAddress(username + "@qq.com"));
                    
                    msg.setRecipient(Message.RecipientType.TO, new InternetAddress(mail_to.getText().toString()));
                    msg.setSubject(mail_topic.getText().toString());
                    msg.setText(mail_content.getText().toString());
                    msg.setSentDate(new Date());

                    Transport.send(msg);
                    handler.sendEmptyMessage(0);
                } catch (MessagingException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                
            }
        }).start();

        System.out.println("Message sent.");

    }

    /**
     * /** 返回
     * 
     * @param v
     */
    public void back(View v) {
        finish();

    }

    /**
     * 返回按钮
     */
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        finish();
        return super.onKeyDown(keyCode, event);
    }

}
``` 

### 完成,以下为另外的例子

### 详细实例

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

































