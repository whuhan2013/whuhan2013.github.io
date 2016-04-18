---
layout: post
title: Android之智能问答机器人
date: 2016-4-18
categories: blog
tags: [android]
description: Android之智能问答机器人
---

### 本文主要利用图灵机器人的接口，所做的一个简单的智能问答机器人  

### 实现  

由于发送与接收消息都是不同的listView，所以要用有两个listVeiw的布局文件  

### 接收消息布局文件  

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/chat_from_createDate"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="2012-09-01 18:30:20"
        style="@style/chat_date_style"
       />

    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:orientation="vertical" >

            <ImageView
                android:id="@+id/chat_from_icon"
                android:layout_width="49dp"
                android:layout_height="49dp"
                android:src="@drawable/icon" />

            <TextView
                android:id="@+id/chat_from_name"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="小貅貅"
                android:textSize="18sp" />
        </LinearLayout>
        
        <TextView 
            android:id="@+id/chat_from_content"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:minHeight="50dp"
            android:background="@drawable/chat_from_msg"
            android:text="有大吗。。。"
            android:textSize="18sp"
            android:textColor="#000"
            android:gravity="center_vertical"
            android:focusable="true"
            android:clickable="true"
            android:lineSpacingExtra="2dp"
            />
    </LinearLayout>

</LinearLayout>
```   


### 发送消息布局文件  


```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/chat_send_createDate"
        style="@style/chat_date_style"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="2012-09-01 18:30:21" />

    <RelativeLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content" >

        <LinearLayout
            android:id="@+id/ly_chat_icon"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:layout_marginLeft="10dp"
            android:orientation="vertical" >

            <ImageView
                android:id="@+id/chat_send_icon"
                android:layout_width="49dp"
                android:layout_height="49dp"
                android:src="@drawable/my" />

            <TextView
                android:id="@+id/chat_send_name"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center"
                android:text="鸿洋"
                android:textSize="18sp" />
        </LinearLayout>

        <TextView
            android:id="@+id/chat_send_content"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_toLeftOf="@id/ly_chat_icon"
            android:background="@drawable/chat_send_msg"
            android:clickable="true"
            android:focusable="true"
            android:gravity="center_vertical"
            android:lineSpacingExtra="2dp"
            android:minHeight="50dp"
            android:text="有大吗。。。"
            android:textColor="#000"
            android:textSize="18sp" />
    </RelativeLayout>

</LinearLayout>
```  


### MainActivity实现  

```
package com.example.android_robot_01;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import android.app.Activity;
import android.content.Context;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.text.TextUtils;
import android.view.View;
import android.view.Window;
import android.view.inputmethod.InputMethodManager;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.Toast;

import com.example.android_robot_01.bean.ChatMessage;
import com.example.android_robot_01.bean.ChatMessage.Type;
import com.zhy.utils.HttpUtils;

public class MainActivity extends Activity
{
    /**
     * 展示消息的listview
     */
    private ListView mChatView;
    /**
     * 文本域
     */
    private EditText mMsg;
    /**
     * 存储聊天消息
     */
    private List<ChatMessage> mDatas = new ArrayList<ChatMessage>();
    /**
     * 适配器
     */
    private ChatMessageAdapter mAdapter;

    private Handler mHandler = new Handler()
    {
        public void handleMessage(android.os.Message msg)
        {
            ChatMessage from = (ChatMessage) msg.obj;
            mDatas.add(from);
            mAdapter.notifyDataSetChanged();
            mChatView.setSelection(mDatas.size() - 1);
        };
    };

    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.main_chatting);
        
        initView();
        
        mAdapter = new ChatMessageAdapter(this, mDatas);
        mChatView.setAdapter(mAdapter);

    }

    private void initView()
    {
        mChatView = (ListView) findViewById(R.id.id_chat_listView);
        mMsg = (EditText) findViewById(R.id.id_chat_msg);
        mDatas.add(new ChatMessage(Type.INPUT, "我是小貅貅，很高兴为您服务"));
    }

    public void sendMessage(View view)
    {
        final String msg = mMsg.getText().toString();
        if (TextUtils.isEmpty(msg))
        {
            Toast.makeText(this, "您还没有填写信息呢...", Toast.LENGTH_SHORT).show();
            return;
        }

        ChatMessage to = new ChatMessage(Type.OUTPUT, msg);
        to.setDate(new Date());
        mDatas.add(to);

        mAdapter.notifyDataSetChanged();
        mChatView.setSelection(mDatas.size() - 1);

        mMsg.setText("");

        // 关闭软键盘
        InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
        // 得到InputMethodManager的实例
        if (imm.isActive())
        {
            // 如果开启
            imm.toggleSoftInput(InputMethodManager.SHOW_IMPLICIT,
                    InputMethodManager.HIDE_NOT_ALWAYS);
            // 关闭软键盘，开启方法相同，这个方法是切换开启与关闭状态的
        }

        new Thread()
        {
            public void run()
            {
                ChatMessage from = null;
                try
                {
                    from = HttpUtils.sendMsg(msg);
                } catch (Exception e)
                {
                    from = new ChatMessage(Type.INPUT, "服务器挂了呢...");
                }
                Message message = Message.obtain();
                message.obj = from;
                mHandler.sendMessage(message);
            };
        }.start();

    }

}
```  

### 消息ListView的adapter，根据类型的不同inflate不同的布局文件   

```
package com.example.android_robot_01;

import java.util.List;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.TextView;

import com.example.android_robot_01.bean.ChatMessage;
import com.example.android_robot_01.bean.ChatMessage.Type;

public class ChatMessageAdapter extends BaseAdapter
{
    private LayoutInflater mInflater;
    private List<ChatMessage> mDatas;

    public ChatMessageAdapter(Context context, List<ChatMessage> datas)
    {
        mInflater = LayoutInflater.from(context);
        mDatas = datas;
    }

    @Override
    public int getCount()
    {
        return mDatas.size();
    }

    @Override
    public Object getItem(int position)
    {
        return mDatas.get(position);
    }

    @Override
    public long getItemId(int position)
    {
        return position;
    }

    /**
     * 接受到消息为1，发送消息为0
     */
    @Override
    public int getItemViewType(int position)
    {
        ChatMessage msg = mDatas.get(position);
        return msg.getType() == Type.INPUT ? 1 : 0;
    }

    @Override
    public int getViewTypeCount()
    {
        return 2;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent)
    {
        ChatMessage chatMessage = mDatas.get(position);

        ViewHolder viewHolder = null;

        if (convertView == null)
        {
            viewHolder = new ViewHolder();
            if (chatMessage.getType() == Type.INPUT)
            {
                convertView = mInflater.inflate(R.layout.main_chat_from_msg,
                        parent, false);
                viewHolder.createDate = (TextView) convertView
                        .findViewById(R.id.chat_from_createDate);
                viewHolder.content = (TextView) convertView
                        .findViewById(R.id.chat_from_content);
                convertView.setTag(viewHolder);
            } else
            {
                convertView = mInflater.inflate(R.layout.main_chat_send_msg,
                        null);

                viewHolder.createDate = (TextView) convertView
                        .findViewById(R.id.chat_send_createDate);
                viewHolder.content = (TextView) convertView
                        .findViewById(R.id.chat_send_content);
                convertView.setTag(viewHolder);
            }

        } else
        {
            viewHolder = (ViewHolder) convertView.getTag();
        }

        viewHolder.content.setText(chatMessage.getMsg());
        viewHolder.createDate.setText(chatMessage.getDateStr());

        return convertView;
    }

    private class ViewHolder
    {
        public TextView createDate;
        public TextView name;
        public TextView content;
    }

}
```  


### 获取消息的工具类,暴露出去的，就是sendMsg这个静态方法，当然将返回的数据也直接封装成了ChatMessage

```
package com.zhy.utils;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.net.HttpURLConnection;
import java.net.URL;
import java.net.URLEncoder;
import java.util.Date;

import com.example.android_robot_01.bean.ChatMessage;
import com.example.android_robot_01.bean.ChatMessage.Type;
import com.example.android_robot_01.bean.CommonException;
import com.example.android_robot_01.bean.Result;
import com.google.gson.Gson;

public class HttpUtils
{
    private static String API_KEY = "534dc342ad15885dffc10d7b5f813451";
    private static String URL = "http://www.tuling123.com/openapi/api";

    /**
     * 发送一个消息，并得到返回的消息
     * @param msg
     * @return
     */
    public static ChatMessage sendMsg(String msg)
    {
        ChatMessage message = new ChatMessage();
        String url = setParams(msg);
        String res = doGet(url);
        Gson gson = new Gson();
        Result result = gson.fromJson(res, Result.class);
        
        if (result.getCode() > 400000 || result.getText() == null
                || result.getText().trim().equals(""))
        {
            message.setMsg("该功能等待开发...");
        }else
        {
            message.setMsg(result.getText());
        }
        message.setType(Type.INPUT);
        message.setDate(new Date());
        
        return message;
    }

    /**
     * 拼接Url
     * @param msg
     * @return
     */
    private static String setParams(String msg)
    {
        try
        {
            msg = URLEncoder.encode(msg, "UTF-8");
        } catch (UnsupportedEncodingException e)
        {
            e.printStackTrace();
        }
        return URL + "?key=" + API_KEY + "&info=" + msg;
    }

    /**
     * Get请求，获得返回数据
     * @param urlStr
     * @return
     */
    private static String doGet(String urlStr)
    {
        URL url = null;
        HttpURLConnection conn = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try
        {
            url = new URL(urlStr);
            conn = (HttpURLConnection) url.openConnection();
            conn.setReadTimeout(5 * 1000);
            conn.setConnectTimeout(5 * 1000);
            conn.setRequestMethod("GET");
            if (conn.getResponseCode() == 200)
            {
                is = conn.getInputStream();
                baos = new ByteArrayOutputStream();
                int len = -1;
                byte[] buf = new byte[128];

                while ((len = is.read(buf)) != -1)
                {
                    baos.write(buf, 0, len);
                }
                baos.flush();
                return baos.toString();
            } else
            {
                throw new CommonException("服务器连接错误！");
            }

        } catch (Exception e)
        {
            e.printStackTrace();
            throw new CommonException("服务器连接错误！");
        } finally
        {
            try
            {
                if (is != null)
                    is.close();
            } catch (IOException e)
            {
                e.printStackTrace();
            }

            try
            {
                if (baos != null)
                    baos.close();
            } catch (IOException e)
            {
                e.printStackTrace();
            }
            conn.disconnect();
        }

    }

}
``` 

### 参考链接:
[Android 智能问答机器人的实现 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/38498353)

### 完成，效果如下  


![这里写图片描述](http://img.blog.csdn.net/20160418090816303) 





























