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

### 更新，增加上一首，下一首功能，并且添加了作者介绍与音乐介绍，代码见github.  

### 使用Fragment作为Item的ViewPager不更新问题.




### 效果如下  

![这里写图片描述](http://img.blog.csdn.net/20160414213754763)  


### 更新

1. 添加了下一首，进度条功能。
2.  解决了使用Fragment作为Item的ViewPager不更新问题.

### 使用Fragment作为Item的ViewPager不更新问题.

在点击下一首时，发现viewPager里面的内容不更新，查找了不少方法，找到解决方案，应该使用FragmentStatePagerAdapter。

我首先使用的是fragmentPagerAdapter.该类内的每一个生成的 Fragment 都将保存在内存之中. 也就是FragmentManager中.所以就算我刷新adapter, 它还是使用的上次缓存的Fragment.  而FragmentStatePagerAdapter的instantiateItem()则会每次都重新创建Fragment. 这样一来就每次就更新了.

### 参考链接
[使用Fragment作为Item的ViewPager不更新问题. - Crazy Bird - 博客频道 - CSDN.NET](http://blog.csdn.net/devilkin64/article/details/39379143)   

[Android-- FragmentStatePagerAdapter分页 - dreamzml的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/dreamzml/article/details/9951577)

### 新代码实现 

```
package com.zj.music;

import android.graphics.Color;
import android.media.AudioManager;
import android.media.MediaPlayer;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.design.widget.CollapsingToolbarLayout;
import android.support.design.widget.TabLayout;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentStatePagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.util.Log;
import android.view.View;
import android.widget.ImageView;
import android.widget.SeekBar;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;
import java.util.Timer;
import java.util.TimerTask;

public class MainActivity extends AppCompatActivity {

    ImageView pause;
    ImageView start;
    ImageView stop;
    ImageView previous;
    ImageView next;
    boolean isPlaying=false;
    String filepath="http://192.168.1.130:8080/music";
    String nowMusic;
    int id=1;
    MediaPlayer mediaPlayer;
    SeekBar song_progress_normal;
    private Timer timer;
    private TimerTask task;
    ViewPager mViewPager;
    MyPagerAdapter adapter;
    TabLayout tabLayout;

    String autors[]=new String[]{"贝多芬","巴赫","舒伯特"};
    String songs[]=new String []{"命运交响曲","G弦上的咏叹调","圣母颂"};

    private static final int UPDATE = 0;
    Handler handler=new Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what)
            {
                case UPDATE:
                    song_progress_normal.setProgress(mediaPlayer.getCurrentPosition());
                    Log.i("mediaplayer",mediaPlayer.getCurrentPosition()+"");
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initTOP();

        pause= (ImageView) findViewById(R.id.pause);
        start= (ImageView) findViewById(R.id.start);
        stop= (ImageView) findViewById(R.id.stop);
        previous= (ImageView) findViewById(R.id.previous);
        next= (ImageView) findViewById(R.id.next);
        song_progress_normal= (SeekBar) findViewById(R.id.song_progress_normal);




        song_progress_normal.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int i, boolean b) {
                int postion = seekBar.getProgress();
                mediaPlayer.seekTo(postion);
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {

            }
        });


        previous.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (id>1)
                {
                    id--;
                    setupViewPager(mViewPager);
                    tabLayout.setupWithViewPager(mViewPager);
                    //adapter.notifyDataSetChanged();
                    stopMusic();
                    startMusic();
                }
            }
        });

        next.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                id++;

                //setupViewPager(mViewPager);
                //adapter.notifyDataSetChanged();
                setupViewPager(mViewPager);

                tabLayout.setupWithViewPager(mViewPager);

                stopMusic();
                startMusic();
            }
        });


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

    private void initTOP() {

        Toolbar mToolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(mToolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);

        setTitle("音乐鉴赏");
        //使用CollapsingToolbarLayout必须把title设置到CollapsingToolbarLayout上，设置到Toolbar上则不会显示
        CollapsingToolbarLayout mCollapsingToolbarLayout = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
        mCollapsingToolbarLayout.setTitle("音乐鉴赏");
        //通过CollapsingToolbarLayout修改字体颜色
        mCollapsingToolbarLayout.setExpandedTitleColor(Color.WHITE);//设置还没收缩时状态下字体颜色
        mCollapsingToolbarLayout.setCollapsedTitleTextColor(Color.BLACK);//设置收缩后Toolbar上字体的颜色


        //设置ViewPager
        mViewPager = (ViewPager) findViewById(R.id.viewpager);
        setupViewPager(mViewPager);

//给TabLayout增加Tab, 并关联ViewPager
         tabLayout = (TabLayout) findViewById(R.id.sliding_tabs);
        tabLayout.addTab(tabLayout.newTab().setText("请您欣赏"));
        tabLayout.addTab(tabLayout.newTab().setText("作者简介"));
        tabLayout.addTab(tabLayout.newTab().setText("歌曲简介"));

        tabLayout.setupWithViewPager(mViewPager);
    }

    private void setupViewPager(ViewPager mViewPager) {
        adapter = new MyPagerAdapter(getSupportFragmentManager());
        adapter.addFragment(DetailFragment.newInstance("请欣赏古典音乐，欲知作者与曲名可右滑"), "请您欣赏");
        adapter.addFragment(DetailFragment.newInstance(autors[id-1]), "作者简介");
        adapter.addFragment(DetailFragment.newInstance(songs[id-1]), "歌曲简介");

        mViewPager.setAdapter(adapter);
    }

     List<Fragment> mFragments;
    static class MyPagerAdapter extends FragmentStatePagerAdapter {
        private  List<Fragment> mFragments=null;
        private  List<String> mFragmentTitles =null;

        public MyPagerAdapter(FragmentManager fm) {
            super(fm);
            mFragments=new ArrayList<>();
            mFragmentTitles = new ArrayList<>();
        }

        public void addFragment(Fragment fragment, String title) {
            mFragments.add(fragment);
            mFragmentTitles.add(title);
        }

        public void removeAllFra()
        {
            mFragments.clear();
            mFragmentTitles.clear();
        }

        @Override
        public Fragment getItem(int position) {
            return mFragments.get(position);
        }

        @Override
        public int getCount() {
            return mFragments.size();
        }

        @Override
        public CharSequence getPageTitle(int position) {
            return mFragmentTitles.get(position);
        }
    }



    private void startMusic() {
        nowMusic=filepath+id+".mp3";
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

                        int max = mediaPlayer.getDuration();
                        Log.i("mediaplayer", max + "最大");
                        song_progress_normal.setMax(max);

                    }
                });
                mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                    @Override
                    public void onCompletion(MediaPlayer mp) {
                        start.setEnabled(true);
                    }
                });

            new Thread(new Runnable() {
                @Override
                public void run() {

                    timer = new Timer();
                    task = new TimerTask() {
                        @Override
                        public void run() {
                            handler.sendEmptyMessage(UPDATE);
                        }
                    };
                    timer.schedule(task, 0, 5000);

                }
            }).start();




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
            timer.cancel();
        }

        start.setEnabled(true);
    }
}
```   


### 效果如下 

![这里写图片描述](http://img.blog.csdn.net/20160416112400646)

### 完成


























