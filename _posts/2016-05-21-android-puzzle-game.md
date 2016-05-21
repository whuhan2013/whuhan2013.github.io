---
layout: post
title: Android拼图游戏
date: 2016-5-21
categories: blog
tags: [自定义控件]
description: Android拼图游戏
---   

### 效果如下

![](http://img.blog.csdn.net/20141029224732877)


### 游戏的设计  
首先我们分析下如何设计这款游戏：          
1、我们需要一个容器，可以放这些图片的块块，为了方便，我们准备使用RelativeLayout配合addRule实现             
2、每个图片的块块，我们准备使用ImageView              
3、点击交换，我们准备使用传统的TranslationAnimation来实现      


### 游戏布局的实现    
首先，我们准备实现能够把一张图片，切成n*n份，放在指定的位置；
我们只需要设置n这个数字，然后根据布局的宽或者高其中的小值，除以n，减去一些边距就可以得到我们ImageView的宽和高了


### 构造方法


```
/** 
     * 设置Item的数量n*n；默认为3 
     */  
    private int mColumn = 3;  
    /** 
     * 布局的宽度 
     */  
    private int mWidth;  
    /** 
     * 布局的padding 
     */  
    private int mPadding;  
    /** 
     * 存放所有的Item 
     */  
    private ImageView[] mGamePintuItems;  
    /** 
     * Item的宽度 
     */  
    private int mItemWidth;  
  
    /** 
     * Item横向与纵向的边距 
     */  
    private int mMargin = 3;  
      
    /** 
     * 拼图的图片 
     */  
    private Bitmap mBitmap;  
    /** 
     * 存放切完以后的图片bean 
     */  
    private List<ImagePiece> mItemBitmaps;  
      
    private boolean once;  
      
    public GamePintuLayout(Context context)  
    {  
        this(context, null);  
    }  
  
    public GamePintuLayout(Context context, AttributeSet attrs)  
    {  
        this(context, attrs, 0);  
    }  
  
    public GamePintuLayout(Context context, AttributeSet attrs, int defStyle)  
    {  
        super(context, attrs, defStyle);  
  
        mMargin = (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,  
                mMargin, getResources().getDisplayMetrics());  
        // 设置Layout的内边距，四边一致，设置为四内边距中的最小值  
        mPadding = min(getPaddingLeft(), getPaddingTop(), getPaddingRight(),  
                getPaddingBottom());  
    }  
```


构造方法里面，我们得到把设置的margin值转化为dp；获得布局的padding值；整体是个正方形，所以我们取padding四个方向中的最小值；
至于margin，作为Item之间的横向与纵向的间距，你喜欢的话可以抽取为自定义属性~~


### onMeasure

```
@Override  
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)  
    {  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);  
      
        // 获得游戏布局的边长  
        mWidth = Math.min(getMeasuredHeight(), getMeasuredWidth());  
  
        if (!once)  
        {  
            initBitmap();  
            initItem();  
        }  
        once = true;  
        setMeasuredDimension(mWidth, mWidth);  
    }  
```         


onMeasure里面主要就是获得到布局的宽度，然后进行图片的准备，以及初始化我们的Item，为Item设置宽度和高度
initBitmap自然就是准备图片了：


```
private void initBitmap()  
    {  
        if (mBitmap == null)  
            mBitmap = BitmapFactory.decodeResource(getResources(),  
                    R.drawable.aa);  
  
        mItemBitmaps = ImageSplitter.split(mBitmap, mColumn);  
  
        Collections.sort(mItemBitmaps, new Comparator<ImagePiece>()  
        {  
            @Override  
            public int compare(ImagePiece lhs, ImagePiece rhs)  
            {  
                return Math.random() > 0.5 ? 1 : -1;  
            }  
        });  
    }  
```


 我们这里如果没有设置mBitmap就准备一张备用图片，然后调用ImageSplitter.split将图片切成n * n 返回一个List<ImagePiece>
切完以后，我们需要将顺序打乱，所以我们调用了sort方法，至于比较器，我们使用random随机比较大小,这样我们就完成了我们的乱序操作，赞不赞~~


```
public class ImageSplitter  
{  
    /** 
     * 将图片切成 , piece *piece 
     *  
     * @param bitmap 
     * @param piece 
     * @return 
     */  
    public static List<ImagePiece> split(Bitmap bitmap, int piece)  
    {  
  
        List<ImagePiece> pieces = new ArrayList<ImagePiece>(piece * piece);  
  
        int width = bitmap.getWidth();  
        int height = bitmap.getHeight();  
  
        Log.e("TAG", "bitmap Width = " + width + " , height = " + height);  
        int pieceWidth = Math.min(width, height) / piece;  
  
        for (int i = 0; i < piece; i++)  
        {  
            for (int j = 0; j < piece; j++)  
            {  
                ImagePiece imagePiece = new ImagePiece();  
                imagePiece.index = j + i * piece;  
                int xValue = j * pieceWidth;  
                int yValue = i * pieceWidth;  
                  
                imagePiece.bitmap = Bitmap.createBitmap(bitmap, xValue, yValue,  
                        pieceWidth, pieceWidth);  
                pieces.add(imagePiece);  
            }  
        }  
        return pieces;  
    }  
}  
```


```
public class ImagePiece  
{  
    public int index = 0;  
    public Bitmap bitmap = null;  
}  
```


没撒说的就是一个根据宽度高度，和n，来切图保存的过程~~
ImagePiece保存的图片以及索引~

图片到此就准备好了，现在看Item的生成已经设置宽高，即initItems


```
private void initItem()  
    {  
        // 获得Item的宽度  
        int childWidth = (mWidth - mPadding * 2 - mMargin * (mColumn - 1))  
                / mColumn;  
        mItemWidth = childWidth;  
  
        mGamePintuItems = new ImageView[mColumn * mColumn];  
        // 放置Item  
        for (int i = 0; i < mGamePintuItems.length; i++)  
        {  
            ImageView item = new ImageView(getContext());  
  
            item.setOnClickListener(this);  
  
            item.setImageBitmap(mItemBitmaps.get(i).bitmap);  
            mGamePintuItems[i] = item;  
            item.setId(i + 1);  
            item.setTag(i + "_" + mItemBitmaps.get(i).index);  
  
            RelativeLayout.LayoutParams lp = new LayoutParams(mItemWidth,  
                    mItemWidth);  
            // 设置横向边距,不是最后一列  
            if ((i + 1) % mColumn != 0)  
            {  
                lp.rightMargin = mMargin;  
            }  
            // 如果不是第一列  
            if (i % mColumn != 0)  
            {  
                lp.addRule(RelativeLayout.RIGHT_OF,//  
                        mGamePintuItems[i - 1].getId());  
            }  
            // 如果不是第一行，//设置纵向边距，非最后一行  
            if ((i + 1) > mColumn)  
            {  
                lp.topMargin = mMargin;  
                lp.addRule(RelativeLayout.BELOW,//  
                        mGamePintuItems[i - mColumn].getId());  
            }  
            addView(item, lp);  
        }  
  
          
    }  
```


可以看到我们的Item宽的计算：childWidth = (mWidth - mPadding * 2 - mMargin * (mColumn - 1) ) / mColumn;
容器的宽度，除去自己的内边距，除去Item间的间距，然后除以Item一行的个数就得到了Item的宽~~
接下来，就是遍历生成Item，根据他们的位置设置Rule，自己仔细看下注释~~
注意两点：
我们为Item设置了setOnClickListener，这个当然，因为我们的游戏就是点Item么~
还有我们为Item设置了Tag：item.setTag(i + "_" + mItemBitmaps.get(i).index);  
tag里面存放了index，也就是正确的位置；还有i，i 可以帮助我们在mItemBitmaps找到当前的Item的图片：（mItemBitmaps.get(i).bitmap）

到此，我们游戏的布局的代码就结束了~~~

然后我们在布局文件里面声明下：


```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="fill_parent"  
    android:layout_height="fill_parent" >  
  
    <com.zhy.gamePintu.view.GamePintuLayout  
        android:id="@+id/id_gameview"  
        android:layout_width="fill_parent"  
        android:layout_height="fill_parent"  
        android:layout_centerInParent="true"  
        android:padding="5dp" >  
    </com.zhy.gamePintu.view.GamePintuLayout>  
  
</RelativeLayout>  
```


Activity里面记得设置这个布局~~           
现在的效果是：


![](http://img.blog.csdn.net/20141029232305603?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG1qNjIzNTY1Nzkx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 游戏的切换效果


1、初步的切换      
还记得我们都给Item添加了onClick的监听么~~
现在我们需要实现，点击两个Item，他们的图片能够发生交换~
那么，我们需要两个成员变量来存储这两个Item，然后再去交换


```
private ImageView mFirst;  
private ImageView mSecond;  
  
@Override  
public void onClick(View v)  
{  
    /** 
     * 如果两次点击是同一个 
     */  
    if (mFirst == v)  
    {  
        mFirst.setColorFilter(null);  
        mFirst = null;  
        return;  
    }  
    //点击第一个Item  
    if (mFirst == null)  
    {  
        mFirst = (ImageView) v;  
        mFirst.setColorFilter(Color.parseColor("#55FF0000"));  
    } else//点击第二个Item  
    {  
        mSecond = (ImageView) v;  
        exchangeView();  
    }  
  
}  
```

点击第一个，通过setColorFilter设置下选中效果，再次点击另一个，那我们就准备调用exchangeView进行交换图片了，当然这个方法我们还没写，先放着~           
如果两次点击同一个，去除选中效果，我们就当什么都没发生。
接下来，我们来实现exchangeView：


```
/** 
     * 交换两个Item的图片 
     */  
    private void exchangeView()  
    {  
                mFirst.setColorFilter(null);  
        String firstTag = (String) mFirst.getTag();  
        String secondTag = (String) mSecond.getTag();  
          
        //得到在list中索引位置  
        String[] firstImageIndex = firstTag.split("_");  
        String[] secondImageIndex = secondTag.split("_");  
          
        mFirst.setImageBitmap(mItemBitmaps.get(Integer  
                .parseInt(secondImageIndex[0])).bitmap);  
        mSecond.setImageBitmap(mItemBitmaps.get(Integer  
                .parseInt(firstImageIndex[0])).bitmap);  
  
        mFirst.setTag(secondTag);  
        mSecond.setTag(firstTag);  
          
        mFirst = mSecond = null;  
  
    }  
```


应该还记得我们之前的setTag吧，忘了，返回去看看，我们还说注意来着~                        
通过getTag，拿到在List中是索引，然后得到bitmap进行交换设置，最后交换tag；                                 
到此我们的交换效果写完了，我们的游戏可以完了~~效果是这样的：

![](http://img.blog.csdn.net/20141029233835115)


### 无缝的动画切换

我们先聊聊怎么添加，我准备使用TranslationAnimation，然后两个Item的top，left也很容器获取；                
但是，要明白，我们实际上，Item只是setImage发生了变化，Item的位置没有变；                           
我们现在需要动画移动效果，比如A移动到B，没问题，移动完成以后，Item得回去吧，但是图片并没有发生变化，我们还是需要手动setImage
这样造成了一个现象，动画切换效果有了，但是最后还是会有一闪，是我们切换图片造成的；                
为了避免上述现象，能够完美的做到切换效果，这里我们引入一个动画图层，专门做动画效果，有点类似ps的图层，下面看我们怎么做；



```
/** 
     * 动画运行的标志位 
     */  
    private boolean isAniming;  
    /** 
     * 动画层 
     */  
    private RelativeLayout mAnimLayout;  
      
    /** 
     * 交换两个Item的图片 
     */  
    private void exchangeView()  
    {  
        mFirst.setColorFilter(null);  
        setUpAnimLayout();  
        // 添加FirstView  
        ImageView first = new ImageView(getContext());  
        first.setImageBitmap(mItemBitmaps  
                .get(getImageIndexByTag((String) mFirst.getTag())).bitmap);  
        LayoutParams lp = new LayoutParams(mItemWidth, mItemWidth);  
        lp.leftMargin = mFirst.getLeft() - mPadding;  
        lp.topMargin = mFirst.getTop() - mPadding;  
        first.setLayoutParams(lp);  
        mAnimLayout.addView(first);  
        // 添加SecondView  
        ImageView second = new ImageView(getContext());  
        second.setImageBitmap(mItemBitmaps  
                .get(getImageIndexByTag((String) mSecond.getTag())).bitmap);  
        LayoutParams lp2 = new LayoutParams(mItemWidth, mItemWidth);  
        lp2.leftMargin = mSecond.getLeft() - mPadding;  
        lp2.topMargin = mSecond.getTop() - mPadding;  
        second.setLayoutParams(lp2);  
        mAnimLayout.addView(second);  
  
        // 设置动画  
        TranslateAnimation anim = new TranslateAnimation(0, mSecond.getLeft()  
                - mFirst.getLeft(), 0, mSecond.getTop() - mFirst.getTop());  
        anim.setDuration(300);  
        anim.setFillAfter(true);  
        first.startAnimation(anim);  
  
        TranslateAnimation animSecond = new TranslateAnimation(0,  
                mFirst.getLeft() - mSecond.getLeft(), 0, mFirst.getTop()  
                        - mSecond.getTop());  
        animSecond.setDuration(300);  
        animSecond.setFillAfter(true);  
        second.startAnimation(animSecond);  
        // 添加动画监听  
        anim.setAnimationListener(new AnimationListener()  
        {  
  
            @Override  
            public void onAnimationStart(Animation animation)  
            {  
                isAniming = true;  
                mFirst.setVisibility(INVISIBLE);  
                mSecond.setVisibility(INVISIBLE);  
            }  
  
            @Override  
            public void onAnimationRepeat(Animation animation)  
            {  
  
            }  
  
            @Override  
            public void onAnimationEnd(Animation animation)  
            {  
                String firstTag = (String) mFirst.getTag();  
                String secondTag = (String) mSecond.getTag();  
  
                String[] firstParams = firstTag.split("_");  
                String[] secondParams = secondTag.split("_");  
  
                mFirst.setImageBitmap(mItemBitmaps.get(Integer  
                        .parseInt(secondParams[0])).bitmap);  
                mSecond.setImageBitmap(mItemBitmaps.get(Integer  
                        .parseInt(firstParams[0])).bitmap);  
  
                mFirst.setTag(secondTag);  
                mSecond.setTag(firstTag);  
                mFirst.setVisibility(VISIBLE);  
                mSecond.setVisibility(VISIBLE);  
                mFirst = mSecond = null;  
                mAnimLayout.removeAllViews();  
                                //checkSuccess();  
                isAniming = false;  
            }  
        });  
  
    }  
  
    /** 
     * 创建动画层 
     */  
    private void setUpAnimLayout()  
    {  
        if (mAnimLayout == null)  
        {  
            mAnimLayout = new RelativeLayout(getContext());  
            addView(mAnimLayout);  
        }  
  
    }  
      
    private int getImageIndexByTag(String tag)  
    {  
        String[] split = tag.split("_");  
        return Integer.parseInt(split[0]);  
  
    }  

```


开始交换时，我们创建一个动画层，然后在这一层上添加上两个一模一样的Item，把原来的Item隐藏了，然后尽情的进行动画切换，setFillAfter为true(表示动画完成后停留在原地)                
动画完毕，我们已经悄悄的把Item的图片交换了，直接显示出来。这样就完美的切换了：                  
大致过程：                 
1、A ，B隐藏                       
2、 A副本动画移动到B的位置；B副本移动到A的位置            
3、A把图片设置为B，把B副本移除，A显示，这样就完美切合了，用户感觉是B移动过去的。                  
4、B同上               

### 注意

动画层的位置和大小是由它其中包含元素的位置所决定的

现在我们的效果：

![](http://img.blog.csdn.net/20141029235338123)




### 游戏胜利的判断


我们在切换完成，进行checkSuccess();的判断；好在我们把图片的正确的顺序存在tag里面~~


```
/** 
     * 判断游戏是否成功 
     */  
    private void checkSuccess()  
    {  
        boolean isSuccess = true;  
        for (int i = 0; i < mGamePintuItems.length; i++)  
        {  
            ImageView first = mGamePintuItems[i];  
            Log.e("TAG", getIndexByTag((String) first.getTag()) + "");  
            if (getIndexByTag((String) first.getTag()) != i)  
            {  
                isSuccess = false;  
            }  
        }  
  
        if (isSuccess)  
        {  
            Toast.makeText(getContext(), "Success , Level Up !",  
                    Toast.LENGTH_LONG).show();  
            // nextLevel();  
        }  
    }  
  
    /** 
     * 获得图片的真正索引 
     * @param tag 
     * @return 
     */  
    private int getIndexByTag(String tag)  
    {  
        String[] split = tag.split("_");  
        return Integer.parseInt(split[1]);  
    }  
```


很简单，遍历所有的Item，根据Tag拿到真正的索引和当然顺序比较，完全一致则胜利~~胜利以后进入下一关。
至于下一关的代码

```
public void nextLevel()  
    {  
        this.removeAllViews();  
        mAnimLayout = null;  
        mColumn++;  
        initBitmap();  
        initItem();  
    }  
```


### 知识点

本文中运用了Collections的sort方法 排序

Comparator是个接口，可重写compare()及equals()这两个方法,用于比价功能；如果是null的话，就是使用元素的默认顺序，如a,b,c,d,e,f,g，就是a,b,c,d,e,f,g这样，当然数字也是这样的。            
compare（a,b）方法:根据第一个参数小于、等于或大于第二个参数分别返回负整数、零或正整数。               
equals（obj）方法：仅当指定的对象也是一个 Comparator，并且强行实施与此 Comparator 相同的排序时才返回 true。          
Collections.sort(list, new PriceComparator());的第二个参数返回一个int型的值，就相当于一个标志，告诉sort方法按什么顺序来对list进行排序。          


这种排序方法是通过mergeSort实现的

### 参考链接

[java中Collections.sort排序详解 - 浮云中的神马 - 博客频道 - CSDN.NET](http://blog.csdn.net/tjcyjd/article/details/6804690)


[Collections的sort方法 排序 - daniel的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/ydd326/article/details/6761447)

### 源代码下载

[源代码](http://download.csdn.net/detail/lmj623565791/8116441)


### 原文链接

[Android 实战美女拼图游戏 你能坚持到第几关 - Hongyang - 博客频道 - CSDN.NET](http://blog.csdn.net/lmj623565791/article/details/40595385)