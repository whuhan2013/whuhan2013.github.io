---
layout: post
title: CSDN客户端实现
date: 2016-4-17
categories: blog
tags: [android]
description: CSDN客户端实现
---

### 本文主要讲解实现了一个CSDN的安卓客户端，主要知识点如下   

1. java爬虫获取网页数据      
2. 将java程序打包成jar包      
3.  Fragment+viewpager+TabPageIndicator实现Tab效果       
4. gestureImageView实现放大缩小图片
5. imageLodar实现异步加载图上   
6. XListView实现下拉刷新   

### java爬虫获取网页资源  


```
package com.zhy.biz;  
  
import java.util.ArrayList;  
import java.util.List;  
  
import org.jsoup.Jsoup;  
import org.jsoup.nodes.Document;  
import org.jsoup.nodes.Element;  
import org.jsoup.select.Elements;  
  
import com.zhy.bean.CommonException;  
import com.zhy.bean.NewsItem;  
import com.zhy.csdn.DataUtil;  
import com.zhy.csdn.URLUtil;  
  
/** 
 * 处理NewItem的业务类 
 * @author zhy 
 *  
 */  
public class NewsItemBiz  
{  
    /** 
     * 业界、移动、云计算 
     *  
     * @param htmlStr 
     * @return 
     * @throws CommonException  
     */  
    public List<NewsItem> getNewsItems( int newsType , int currentPage) throws CommonException  
    {  
        String urlStr = URLUtil.generateUrl(newsType, currentPage);  
          
        String htmlStr = DataUtil.doGet(urlStr);  
          
        List<NewsItem> newsItems = new ArrayList<NewsItem>();  
        NewsItem newsItem = null;  
  
        Document doc = Jsoup.parse(htmlStr);  
        Elements units = doc.getElementsByClass("unit");  
        for (int i = 0; i < units.size(); i++)  
        {  
            newsItem = new NewsItem();  
            newsItem.setNewsType(newsType);  
  
            Element unit_ele = units.get(i);  
  
            Element h1_ele = unit_ele.getElementsByTag("h1").get(0);  
            Element h1_a_ele = h1_ele.child(0);  
            String title = h1_a_ele.text();  
            String href = h1_a_ele.attr("href");  
  
            newsItem.setLink(href);  
            newsItem.setTitle(title);  
  
            Element h4_ele = unit_ele.getElementsByTag("h4").get(0);  
            Element ago_ele = h4_ele.getElementsByClass("ago").get(0);  
            String date = ago_ele.text();  
  
            newsItem.setDate(date);  
  
            Element dl_ele = unit_ele.getElementsByTag("dl").get(0);// dl  
            Element dt_ele = dl_ele.child(0);// dt  
            try  
            {// 可能没有图片  
                Element img_ele = dt_ele.child(0);  
                String imgLink = img_ele.child(0).attr("src");  
                newsItem.setImgLink(imgLink);  
            } catch (IndexOutOfBoundsException e)  
            {  
  
            }  
            Element content_ele = dl_ele.child(1);// dd  
            String content = content_ele.text();  
            newsItem.setContent(content);  
            newsItems.add(newsItem);  
        }  
  
        return newsItems;  
  
    }  
  
}  
```   


### 详细实现参见链接   

[抓取csdn上的各类别的文章 （制作csdn app 二） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/23532797)

### 将java项目打包成jar包  

参考链接：[在Eclipse中将Java项目打包为jar - 闵开慧的个人页面 - 开源中国社区](http://my.oschina.net/mkh/blog/265542?fromerr=98uL1b2Y)


### Fragment+viewpager+TabPageIndicator实现Tab效果   


```
package com.zhy.csdndemo;  
  
  
import com.viewpagerindicator.TabPageIndicator;  
  
  
import android.os.Bundle;  
import android.support.v4.app.FragmentActivity;  
import android.support.v4.app.FragmentPagerAdapter;  
import android.support.v4.view.ViewPager;  
  
  
public class MainActivity extends FragmentActivity  
{  
    private TabPageIndicator mIndicator ;  
    private ViewPager mViewPager ;  
    private FragmentPagerAdapter mAdapter ;  
  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
      
        mIndicator = (TabPageIndicator) findViewById(R.id.id_indicator);  
        mViewPager = (ViewPager) findViewById(R.id.id_pager);  
        mAdapter = new TabAdapter(getSupportFragmentManager());  
        mViewPager.setAdapter(mAdapter);  
        mIndicator.setViewPager(mViewPager, 0);  
          
          
          
    }  
      
      
  
  
}  
```

### 详细实现参见链接  

[Android 使用Fragment，ViewPagerIndicator 制作csdn app主要框架 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/23513993)


### viewPager的适配器 

```
package com.zhy.csdndemo;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;

public class TabAdapter extends FragmentPagerAdapter
{

    public static final String[] TITLES = new String[] { "业界", "移动", "研发", "程序员杂志", "云计算" };

    public TabAdapter(FragmentManager fm)
    {
        super(fm);
    }

    @Override
    public Fragment getItem(int arg0)
    {
        MainFragment fragment = new MainFragment(arg0+1);
        return fragment;
    }

    @Override
    public CharSequence getPageTitle(int position)
    {
        return TITLES[position % TITLES.length];
    }

    @Override
    public int getCount()
    {
        return TITLES.length;
    }

}
```  


### viewPager中的Fragment实现，其中用xListView实现了获取数据与刷新等功能  

```
package com.zhy.csdndemo;

import java.util.ArrayList;
import java.util.List;

import me.maxwin.view.IXListViewLoadMore;
import me.maxwin.view.IXListViewRefreshListener;
import me.maxwin.view.XListView;
import android.annotation.SuppressLint;
import android.os.AsyncTask;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.zhy.bean.CommonException;
import com.zhy.bean.NewsItem;
import com.zhy.biz.NewsItemBiz;
import com.zhy.csdn.Constaint;
import com.zhy.csdndemo.adapter.NewsItemAdapter;

@SuppressLint("ValidFragment")
public class MainFragment extends Fragment implements IXListViewRefreshListener, IXListViewLoadMore
{
    /**
     * 默认的newType
     */
    private int newsType = Constaint.NEWS_TYPE_YEJIE;
    /**
     * 当前页面
     */
    private int currentPage = 1;
    /**
     * 处理新闻的业务类
     */
    private NewsItemBiz mNewsItemBiz;
    /**
     * 扩展的ListView
     */
    private XListView mXListView;
    /**
     * 数据适配器
     */
    private NewsItemAdapter mAdapter;
    
    /**
     * 数据
     */
    private List<NewsItem> mDatas = new ArrayList<NewsItem>();

    /**
     * 获得newType
     * @param newsType
     */
    public MainFragment(int newsType)
    {
        this.newsType = newsType;
        mNewsItemBiz = new NewsItemBiz();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState)
    {
        return inflater.inflate(R.layout.tab_item_fragment_main, null);
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState)
    {
        super.onActivityCreated(savedInstanceState);
        mAdapter = new NewsItemAdapter(getActivity(), mDatas);
        /**
         * 初始化
         */
        mXListView = (XListView) getView().findViewById(R.id.id_xlistView);
        mXListView.setAdapter(mAdapter);
        mXListView.setPullRefreshEnable(this);
        mXListView.setPullLoadEnable(this);
        //mXListView.NotRefreshAtBegin();
        /**
         * 进来时直接刷新
         */
        mXListView.startRefresh();
    }

    @Override
    public void onRefresh()
    {
        new LoadDatasTask().execute();
    }

    @Override
    public void onLoadMore()
    {
        // TODO Auto-generated method stub

    }
    /**
     * 记载数据的异步任务
     * @author zhy
     *
     */
    class LoadDatasTask extends AsyncTask<Void, Void, Void>
    {

        @Override
        protected Void doInBackground(Void... params)
        {
            try
            {
                List<NewsItem> newsItems = mNewsItemBiz.getNewsItems(newsType, currentPage);
                mDatas = newsItems;
            } catch (CommonException e)
            {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }

            return null;
        }

        @Override
        protected void onPostExecute(Void result)
        {
            mAdapter.addAll(mDatas);
            mAdapter.notifyDataSetChanged();
            mXListView.stopRefresh();
        }

    }

}
```   

### xListView的adapter,其中使用了imageLoder

```
package com.zhy.csdndemo.adapter;

import java.util.List;

import android.content.Context;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import com.nostra13.universalimageloader.core.DisplayImageOptions;
import com.nostra13.universalimageloader.core.ImageLoader;
import com.nostra13.universalimageloader.core.ImageLoaderConfiguration;
import com.nostra13.universalimageloader.core.display.FadeInBitmapDisplayer;
import com.nostra13.universalimageloader.core.display.RoundedBitmapDisplayer;
import com.zhy.bean.NewsItem;
import com.zhy.csdn.DataUtil;
import com.zhy.csdndemo.R;

public class NewsItemAdapter extends BaseAdapter
{

    private LayoutInflater mInflater;
    private List<NewsItem> mDatas;
    
    /**
     * 使用了github开源的ImageLoad进行了数据加载
     */
    private ImageLoader imageLoader = ImageLoader.getInstance();
    private DisplayImageOptions options;

    public NewsItemAdapter(Context context, List<NewsItem> datas)
    {
        this.mDatas = datas;
        mInflater = LayoutInflater.from(context);
        
        imageLoader.init(ImageLoaderConfiguration.createDefault(context));
        options = new DisplayImageOptions.Builder().showStubImage(R.drawable.images)
                .showImageForEmptyUri(R.drawable.images).showImageOnFail(R.drawable.images).cacheInMemory()
                .cacheOnDisc().displayer(new RoundedBitmapDisplayer(20)).displayer(new FadeInBitmapDisplayer(300))
                .build();
    }

    public void addAll(List<NewsItem> mDatas)
    {
        this.mDatas.addAll(mDatas);
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

    @Override
    public View getView(int position, View convertView, ViewGroup parent)
    {
        ViewHolder holder = null;
        if (convertView == null)
        {
            convertView = mInflater.inflate(R.layout.news_item_yidong, null);
            holder = new ViewHolder();

            holder.mContent = (TextView) convertView.findViewById(R.id.id_content);
            holder.mTitle = (TextView) convertView.findViewById(R.id.id_title);
            holder.mDate = (TextView) convertView.findViewById(R.id.id_date);
            holder.mImg = (ImageView) convertView.findViewById(R.id.id_newsImg);

            convertView.setTag(holder);
        } else
        {
            holder = (ViewHolder) convertView.getTag();
        }
        NewsItem newsItem = mDatas.get(position);
        holder.mTitle.setText(DataUtil.ToDBC(newsItem.getTitle()));
        holder.mContent.setText(newsItem.getContent());
        holder.mDate.setText(newsItem.getDate());
        if (newsItem.getImgLink() != null)
        {
            holder.mImg.setVisibility(View.VISIBLE);
            imageLoader.displayImage(newsItem.getImgLink(), holder.mImg, options);
        } else
        {
            holder.mImg.setVisibility(View.GONE);
        }

        return convertView;
    }

    private final class ViewHolder
    {
        TextView mTitle;
        TextView mContent;
        ImageView mImg;
        TextView mDate;
    }

}

```   


### 参考链接：
[客户端上显示csdn上的各类别下的的文章列表 （制作csdn app 三） - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/23597229)

### 效果如下
![这里写图片描述](http://img.blog.csdn.net/20160417202355024)


























