---
layout: post
title: Android实现网络音乐播放器
date: 2016-4-14
categories: blog
tags: [android]
description: Android实现网络音乐播放器
---

### 本文是一个简单的音乐播放器  

### 布局代码  

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.zj.music.MainActivity">

    <ProgressBar
        android:id="@+id/song_progress_normal"
        style="@style/Widget.AppCompat.ProgressBar.Horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="top"
        android:maxHeight="5dp"
        android:progress="30"
        android:tag="tint_accent_color" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/song_progress_normal"
        android:orientation="horizontal"
        android:gravity="center_horizontal"
        >
        <ImageView
            android:id="@+id/previous"
            android:clickable="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/btn_playback_previous"/>
        <ImageView
            android:layout_marginLeft="20dp"
            android:id="@+id/start"
            android:clickable="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/start"/>
        <ImageView
            android:id="@+id/pause"
            android:clickable="true"
            android:layout_marginLeft="20dp"
            android:layout_marginRight="20dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            />
        <ImageView
            android:layout_marginRight="20dp"
            android:id="@+id/stop"
            android:clickable="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/stop"/>
        <ImageView
            android:id="@+id/next"
            android:clickable="true"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/btn_playback_next"/>

    </LinearLayout>
</RelativeLayout>
```   

### java代码实现  

```
package com.zj.music;

import android.media.AudioManager;
import android.media.MediaPlayer;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    ImageView pause;
    ImageView start;
    ImageView stop;
    ImageView previous;
    ImageView next;
    boolean isPlaying=false;
    String filepath="http://10.129.69.114:8080/music";
    MediaPlayer mediaPlayer;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        pause= (ImageView) findViewById(R.id.pause);
        start= (ImageView) findViewById(R.id.start);
        stop= (ImageView) findViewById(R.id.stop);
        pause.setBackgroundResource(R.drawable.btn_playback_pause);

        start.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startMusic();
            }
        });

        stop.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                        stopMusic();
            }
        });

        pause.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                isPlaying=!isPlaying;
                System.out.println("1243544444444");
                if (isPlaying)
                {
                    pause.setBackgroundResource(R.drawable.btn_playback_pause);
                    rePlayMusic();
                }else
                {
                    pause.setBackgroundResource(R.drawable.btn_playback_play);
                    pauseMusic();
                }
            }
        });
    }

    private void startMusic() {
       String nowMusic=filepath+"1.mp3";
        try {

                mediaPlayer = new MediaPlayer();

                mediaPlayer.setDataSource(nowMusic);//设置播放的数据源。
                mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
                //mediaPlayer.prepare();//同步的准备方法。
                mediaPlayer.prepareAsync();//异步的准备
                mediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                    @Override
                    public void onPrepared(MediaPlayer mp) {
                        mediaPlayer.start();
                        start.setEnabled(false);
                    }
                });
                mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                    @Override
                    public void onCompletion(MediaPlayer mp) {
                        start.setEnabled(true);
                    }
                });

        } catch (Exception e) {
            e.printStackTrace();
            Toast.makeText(this, "播放失败", Toast.LENGTH_SHORT).show();
        }
    }

    private void pauseMusic() {

        if(mediaPlayer!=null)
        mediaPlayer.pause();

    }

    private void rePlayMusic()
    {
        if(mediaPlayer!=null)
        mediaPlayer.start();
    }
    public void stopMusic() {
        if(mediaPlayer!=null&&mediaPlayer.isPlaying()){
            mediaPlayer.stop();
            mediaPlayer.release();
            mediaPlayer = null;
        }

        start.setEnabled(true);
    }
}
```  


### 注意，音乐文件应该放在webapp/root文件下，而不能直接放在根目录下，否则读不到。 


### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160414213754763)


























