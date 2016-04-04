---
layout: post
title: Android之ScrollView
date: 2016-4-4
categories: blog
tags: [android]
description: Android之ScrollView
---

  1、ScrollView和HorizontalScrollView是为控件或者布局添加滚动条  

2、上述两个控件只能有一个孩子，但是它并不是传统意义上的容器

3、上述两个控件可以互相嵌套

4、滚动条的位置现在的实验结果是：可以由layout_width和layout_height设定

5、ScrollView用于设置垂直滚动条，HorizontalScrollView用于设置水平滚动条：需要注意的是，有一个属性是    scrollbars 可以设置滚动条的方向:但是ScrollView设置成horizontal是和设置成none是效果同，HorizontalScrollView设置成vertical和none的效果同。  


### ScrollView嵌套listView时，listView会产生只能显示一行的状况，可以用如下方法解决。  


### 解决方法

### 方法1：手动设置listView高度  

```
01.public class MainActivity extends Activity {   
 02.    private ListView listView;   
 03.    @Override   
 04.    protected void onCreate(Bundle savedInstanceState) {   
 05.        super.onCreate(savedInstanceState);   
 06.        setContentView(R.layout.activity_main);   
 07.        listView = (ListView) findViewById(R.id.listView1);   
 08.        String[] adapterData = new String[] { "Afghanistan", "Albania",… … "Bosnia"};   
 09.        listView.setAdapter(new ArrayAdapter<String>(this,android.R.layout.simple_list_item_1,adapterData));   
 10.        setListViewHeightBasedOnChildren(listView);   
 11.    }   
 12.    public void setListViewHeightBasedOnChildren(ListView listView) {   
 13.        // 获取ListView对应的Adapter   
 14.        ListAdapter listAdapter = listView.getAdapter();   
 15.        if (listAdapter == null) {   
 16.            return;   
 17.        }   
 18.   
 19.        int totalHeight = 0;   
 20.        for (int i = 0, len = listAdapter.getCount(); i < len; i++) {   
 21.            // listAdapter.getCount()返回数据项的数目   
 22.            View listItem = listAdapter.getView(i, null, listView);   
 23.            // 计算子项View 的宽高   
 24.            listItem.measure(0, 0);    
 25.            // 统计所有子项的总高度   
 26.            totalHeight += listItem.getMeasuredHeight();    
 27.        }   
 28.   
 29.        ViewGroup.LayoutParams params = listView.getLayoutParams();   
 30.        params.height = totalHeight+ (listView.getDividerHeight() * (listAdapter.getCount() - 1));   
 31.        // listView.getDividerHeight()获取子项间分隔符占用的高度   
 32.        // params.height最后得到整个ListView完整显示需要的高度   
 33.        listView.setLayoutParams(params);   
 34.    }   
 35.}   
``` 

### 方法2：使用单个ListView取代ScrollView中所有内容 

就是说，把整个需要放在ScrollView中的内容，统统放在ListView中，原ListView上方的数据和下方数据，都作为现ListView的一个itemView，和原ListView中的单条数据是平级的关系。

```
public View getView(int position, View convertView, ViewGroup parent) {
            //列表第一项
    if(position == 0){
       convertView = inflater.inflate(R.layout.item_solution2_top, null);
        return convertView;
    }
            //列表最后一项
    else if(position == 21){
        convertView = inflater.inflate(R.layout.item_solution2_bottom, null);
        return convertView;
    }
            //普通列表项
    ViewHolder h = null;
    if(convertView == null || convertView.getTag() == null){
        convertView = inflater.inflate(R.layout.item_listview_data, null);
        h = new ViewHolder();
        h.tv = (TextView) convertView.findViewById(R.id.item_listview_data_tv);
        convertView.setTag(h);
    }else{
        h = (ViewHolder) convertView.getTag();
    }
    h.tv.setText("第"+ position + "条数据");
    return convertView;
}
``` 

### 方法3：使用LinearLayout取代ListView

    既然ListView不能适应ScrollView，那就换一个可以适应ScrollView的控件，干嘛非要吊死在ListView这一棵树上呢？而LinearLayout是最好的选择。但如果我仍想继续使用已经定义好的Adater呢？我们只需要自定义一个类继承自LinearLayout，为其加上对BaseAdapter的适配。 

```
/**
* 取代ListView的LinearLayout，使之能够成功嵌套在ScrollView中
* @author terry_龙
*/
public class LinearLayoutForListView extends LinearLayout {
    private BaseAdapter adapter;
    private OnClickListener onClickListener = null;
    /**
     * 绑定布局
     */
    public void bindLinearLayout() {
        int count = adapter.getCount();
        this.removeAllViews();
        for (int i = 0; i < count; i++) {
            View v = adapter.getView(i, null, null);
            v.setOnClickListener(this.onClickListener);
            addView(v, i);
        }
       Log.v("countTAG", "" + count);
    }
    public LinearLayoutForListView(Context context) {
        super(context);
        }
```

上面的代码拷贝保存为LinearLayoutForListView.class，或者直接拷贝Demo中的这个类在自己的工程里。我们只需要把原来xml布局文件中的ListView替换为这个类就行了：

```
<pm.nestificationbetweenscrollviewandabslistview.mywidgets.LinearLayoutForListView
    android:id="@+id/act_solution_3_mylinearlayout"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical" >
</pm.nestificationbetweenscrollviewandabslistview.mywidgets.LinearLayoutForListView>
```


在Activity中也把ListView改成LinearLayoutForListView，就能成功运行了。

```
mylinearlayout = (LinearLayoutForListView) findViewById(R.id.act_solution_3_mylinearlayout);
adapter = new AdapterForListView(this);
mylinearlayout.setAdapter(adapter);
```

### 方法四  

自定义可适应ScrollView的ListView
    这个方法和上面的方法是异曲同工，方法3是自定义了LinearLayout以取代ListView的功能，但如果我脾气就是倔，就是要用ListView怎么办？那就只好自定义一个类继承自ListView，通过重写其onMeasure方法，达到对ScrollView适配的效果。
    下面是继承了ListView的自定义类：



```
import android.content.Context;
import android.util.AttributeSet;
import android.widget.ListView;
public class ListViewForScrollView extends ListView {
    public ListViewForScrollView(Context context) {
        super(context);
    }
    public ListViewForScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public ListViewForScrollView(Context context, AttributeSet attrs,
        int defStyle) {
        super(context, attrs, defStyle);
    }
    @Override
    /**
     * 重写该方法，达到使ListView适应ScrollView的效果
     */
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2,
        MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}
```

三个构造方法完全不用动，只要重写onMeasure方法，需要改动的地方比起方法3少了不是一点半点…
    在xml布局中和Activty中使用的ListView改成这个自定义ListView就行了。代码就省了吧…
    这个方法和方法1有一个同样的毛病，就是默认显示的首项是ListView，需要手动把ScrollView滚动至最顶端。

```
sv = (ScrollView) findViewById(R.id.act_solution_4_sv);
sv.smoothScrollTo(0, 0);
```

### 参考链接   

[日积月累：ScrollView嵌套ListView只显示一行 - 郑文亮 - 博客园](http://www.cnblogs.com/zhwl/p/3333585.html)

[四种方案解决ScrollView嵌套ListView问题 - Android开发者交流 - 安卓论坛](http://bbs.anzhuo.cn/thread-982250-1-1.html)

### 完成 ,效果如下

![这里写图片描述](http://img.blog.csdn.net/20160404192323397)















