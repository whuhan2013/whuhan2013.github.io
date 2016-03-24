---
layout: post
title: Android之activity框架
date: 2016-3-24
categories: blog
tags: [android]
description: 通告一下，我已不再每天写千字文，准备采用以下的方法进行练习，由于文章篇幅较长，链接较多，建议到简书或博客进行阅读。
---

### 在安卓应用中，经常需要Activity中经常需要有大量相似的Activity类，这些类往往有相似的结构与功能，因此产生了大量重复代码，为此，以下提供一种方法有效的降低了代码冗余。

## 定义Activity工具类
```
 *      应用程序Activity管理类：用于Activity管理和应用程序退出
 * 修订历史 ：
 * 
 * ============================================================
 **/

public class AppManager {
  
  private static Stack<Activity> activityStack;
  private static AppManager instance;
  
  private AppManager(){}
  /**
   * 单一实例
   */
  public static AppManager getAppManager(){
    if(instance==null){
      instance=new AppManager();
    }
    return instance;
  }
  /**
   * 添加Activity到堆栈
   */
  public void addActivity(Activity activity){
    if(activityStack==null){
      activityStack=new Stack<Activity>();
    }
    activityStack.add(activity);
  }
  /**
   * 获取当前Activity（堆栈中最后一个压入的）
   */
  public Activity currentActivity(){
    Activity activity=activityStack.lastElement();
    return activity;
  }
  /**
   * 结束当前Activity（堆栈中最后一个压入的）
   */
  public void finishActivity(){
    Activity activity=activityStack.lastElement();
    finishActivity(activity);
  }
  /**
   * 结束指定的Activity
   */
  public void finishActivity(Activity activity){
    if(activity!=null){
      activityStack.remove(activity);
      activity.finish();
      activity=null;
    }
  }
  /**
   * 结束指定类名的Activity
   */
  public void finishActivity(Class<?> cls){
    for (Activity activity : activityStack) {
      if(activity.getClass().equals(cls) ){
        finishActivity(activity);
      }
    }
  }
  /**
   * 结束所有Activity
   */
  public void finishAllActivity(){
    for (int i = 0, size = activityStack.size(); i < size; i++){
            if (null != activityStack.get(i)){
              activityStack.get(i).finish();
            }
      }
    activityStack.clear();
  }
  /**
   * 退出应用程序
   */
  public void AppExit(Context context) {
    try {
      finishAllActivity();
      ActivityManager activityMgr= (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
      activityMgr.restartPackage(context.getPackageName());
      System.exit(0);
      android.os.Process.killProcess(android.os.Process.myPid());
    } catch (Exception e) { }
  }
}
```

## 定义Activity基类
```
public abstract class BaseActivity extends Activity implements OnClickListener {
  /**
   * Android生命周期回调方法-创建
   */
  @Override
  public void onCreate(Bundle paramBundle) {
    super.onCreate(paramBundle);
    // 设置没有标题
    // requestWindowFeature(Window.FEATURE_NO_TITLE);
    mContext = this;
    app = (AmbowApplication) getApplication();
    AppManager.getAppManager().addActivity(this);
    initView();
  }

  /**
   * Android生命周期回调方法-销毁
   */
  @Override
  protected void onDestroy() {
    AppManager.getAppManager().finishActivity(this);
    super.onDestroy();

  }

  @Override
  protected void onResume() {

    super.onResume();
    overridePendingTransition(android.R.anim.fade_in,
        android.R.anim.fade_out);
  }

  @Override
  protected void onPause() {
    super.onPause();
  }
  /**
   * 初始化界面
   */
  private void initView() {
    loadViewLayout();
    findViewById();
    processLogic();
    setListener();
  }

/**
   * find控件
   */
  protected abstract void findViewById();

  /**
   * 加载布局
   */
  protected abstract void loadViewLayout();

  /**
   * 后台获取数据
   */
  protected abstract void processLogic();

  /**
   * 设置监听
   */
  protected abstract void setListener();
```

### 将获取布局，获取View,获取后台数据，设置监听设置为抽象方法，使得子类继承时必须要实现。

## 子类对抽象方法的实现
```
  @Override
  protected void findViewById() {
    newsLv = (ListView) this.findViewById(R.id.news_lv);
    gallery = (MyGallery) galleryView.findViewById(R.id.gallery);
    galleryRl = (RelativeLayout) galleryView.findViewById(R.id.rl_gallery);
    bannerTv = (TextView) galleryView.findViewById(R.id.banner_tv);
    addMoreBtn = (TextView) addMoreView.findViewById(R.id.btn_add_more);

  }

  @Override
  protected void loadViewLayout() {
    setContentView(R.layout.news_list_layout);
    galleryView = View.inflate(mContext, R.layout.gallery_layout, null);
    addMoreView = View.inflate(mContext, R.layout.add_more, null);
    setTitleBarView(false, "资讯", -1, true);
  }

  @Override
  protected void processLogic() {
    newsLv.addHeaderView(galleryView);
    eduNewsList = new ArrayList<NewsListEntity.News>();
    newsLv.addFooterView(addMoreView);
//    getTopNewsData();

  }

  @Override
  protected void setListener() {
    addMoreBtn.setOnClickListener(this);
    newsLv.setOnItemClickListener(new OnItemClickListener() {

      @Override
      public void onItemClick(AdapterView<?> arg0, View arg1, int arg2,
          long arg3) {
        Intent detailIntent = new Intent(mContext,
            NewsDetailActivity.class);
        if (eduNewsList.size() > arg2 - 1) {
          detailIntent.putExtra("id", eduNewsList.get(arg2 - 1).Id);
          startActivity(detailIntent);
        }

      }
    });
```













