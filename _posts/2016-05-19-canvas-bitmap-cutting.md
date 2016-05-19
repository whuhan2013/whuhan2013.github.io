---
layout: post
title: Android下利用Bitmap切割图片
date: 2016-5-19
categories: blog
tags: [android]
description: Android下利用Bitmap切割图片
---   

在自己自定义的一个组件中由于需要用图片显示数字编号，而当前图片就只有一张，上面有0-9是个数字，于是不得不考虑将其中一个个的数字切割下来，需要显示什么数字，只需要组合一下就好了。 

下面是程序的关键代码： 

在MyView（继承于View）类中的重写的onDraw(Canvas canvas)方法中，有如下代码段：

```
Bitmap resource = BitmapFactory.decodeResource(this.getResources(), R.drawable.num);  
        Bitmap zero = Bitmap.createBitmap(resource, 0, 0, 12, 12);  
        Bitmap one = Bitmap.createBitmap(resource, 12, 0, 12, 12);  
        Bitmap two = Bitmap.createBitmap(resource, 24, 0, 12, 12);  
        Bitmap three = Bitmap.createBitmap(resource, 36, 0, 12, 12);  
        Bitmap four = Bitmap.createBitmap(resource, 48, 0, 12, 12);  
        Bitmap five = Bitmap.createBitmap(resource, 60, 0, 12, 12);  
        Bitmap six = Bitmap.createBitmap(resource, 72, 0, 12, 12);  
        Bitmap seven = Bitmap.createBitmap(resource, 84, 0, 12, 12);  
        Bitmap eight = Bitmap.createBitmap(resource, 96, 0, 12, 12);  
        Bitmap nine = Bitmap.createBitmap(resource, 108, 0, 12, 12);
```



其中R.drawable.num为数字图片，每个数字占据的像素为12*12，Bitmap.createBitmap方法中的五个参数意义分别为：需要切割的图片资源、切割起始点的X坐标、切割起始点的Y坐标、切割多宽、切割多高。 
    切割下来之后就非常简单的就可以显示各种数字了，例如：用String类型的number表示需要显示的数字，则

```
char nums[] = number.toCharArray();  
        for(int i = 0; i < nums.length; i ++) {  
            if(nums[i] == '0') {  
                canvas.drawBitmap(zero, i * 12, 0, mPaint);  
            } else if(nums[i] == '1') {  
                canvas.drawBitmap(one, i * 12, 0, mPaint);  
            } else if(nums[i] == '2') {  
                canvas.drawBitmap(two, i * 12, 0, mPaint);  
            } else if(nums[i] == '3') {  
                canvas.drawBitmap(three, i * 12, 0, mPaint);  
            } else if(nums[i] == '4') {  
                canvas.drawBitmap(four, i * 12, 0, mPaint);  
            } else if(nums[i] == '5') {  
                canvas.drawBitmap(five, i * 12, 0, mPaint);  
            } else if(nums[i] == '6') {  
                canvas.drawBitmap(six, i * 12, 0, mPaint);  
            } else if(nums[i] == '7') {  
                canvas.drawBitmap(seven, i * 12, 0, mPaint);  
            } else if(nums[i] == '8') {  
                canvas.drawBitmap(eight, i * 12, 0, mPaint);  
            } else if(nums[i] == '9') {  
                canvas.drawBitmap(nine, i * 12, 0, mPaint);  
            }  
        }  
```

  其中canvas为画布，drawBitmap方法中的四个参数的意义分别为：需要绘制的图片资源、在画布上绘制的起始点的X坐标、Y坐标、画笔。其中画笔在此处可以不进行任何设置，只需new一个出来即可，Paint mPaint = new Paint();

### 完整代码

### NumView


```
package com.barney;

import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Paint;
import android.graphics.Rect;
import android.util.DisplayMetrics;
import android.view.View;

public class NumView extends View {
    private static Paint mPaint;
    private String num;

    public NumView(Context context, String num) {
        super(context);

        this.num = num;
        mPaint = new Paint();
    }

    @Override
    public void draw(Canvas canvas) {
        super.onDraw(canvas);
        
        int base = 0;
        mPaint.setAntiAlias(true);
        
        DisplayMetrics dm = new DisplayMetrics();  
        dm = getResources().getDisplayMetrics(); 
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inDensity = dm.densityDpi;
        
        Bitmap resource = BitmapFactory.decodeResource(this.getResources(), R.drawable.num, options);
        Bitmap zero = Bitmap.createBitmap(resource, 0, 0, 12, 12);
        Bitmap one = Bitmap.createBitmap(resource, 12, 0, 12, 12);
        Bitmap two = Bitmap.createBitmap(resource, 24, 0, 12, 12);
        Bitmap three = Bitmap.createBitmap(resource, 36, 0, 12, 12);
        Bitmap four = Bitmap.createBitmap(resource, 48, 0, 12, 12);
        Bitmap five = Bitmap.createBitmap(resource, 60, 0, 12, 12);
        Bitmap six = Bitmap.createBitmap(resource, 72, 0, 12, 12);
        Bitmap seven = Bitmap.createBitmap(resource, 84, 0, 12, 12);
        Bitmap eight = Bitmap.createBitmap(resource, 96, 0, 12, 12);
        Bitmap nine = Bitmap.createBitmap(resource, 108, 0, 12, 12);
        
        char nums[] = num.toCharArray();
        for(int i = 0; i < nums.length; i ++) {
            Rect rect = new Rect();
            rect.set(base + i * 12, 0, base + i * 12 + 12, 12);
            Bitmap bitmap = null;
            if(nums[i] == '0') {
                bitmap = zero;
            } else if(nums[i] == '1') {
                bitmap = one;
            } else if(nums[i] == '2') {
                bitmap = two;
            } else if(nums[i] == '3') {
                bitmap = three;
            } else if(nums[i] == '4') {
                bitmap = four;
            } else if(nums[i] == '5') {
                bitmap = five;
            } else if(nums[i] == '6') {
                bitmap = six;
            } else if(nums[i] == '7') {
                bitmap = seven;
            } else if(nums[i] == '8') {
                bitmap = eight;
            } else if(nums[i] == '9') {
                bitmap = nine;
            }
            
            canvas.drawBitmap(bitmap,null, rect, mPaint);
        }
    }
}
```


### BitmapDemoActivity


```
package com.barney;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.LinearLayout;

public class BitmapDemoActivity extends Activity {
    
    private EditText myEditText;
    private Button myButton;
    private LinearLayout myLinearLayout;
    
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        
        myButton = (Button) this.findViewById(R.id.myButton);
        myEditText = (EditText) this.findViewById(R.id.myEditText);
        myLinearLayout = (LinearLayout) this.findViewById(R.id.myLinearLayout);
        myButton.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                // TODO Auto-generated method stub
                String num = myEditText.getText().toString();
                NumView numView = new NumView(BitmapDemoActivity.this, num);
                myLinearLayout.removeAllViews();
                myLinearLayout.addView(numView);
            }
            
        });
    }
}
```


### 源代码

[源代码](http://dl.iteye.com/topics/download/2bd62f21-86af-3759-a24f-de4002593751)


### 参考链接

[Android下利用Bitmap切割图片 - - ITeye技术网站](http://chen592969029.iteye.com/blog/749100)


### 效果如下

![这里写图片描述](http://img.blog.csdn.net/20160519143931165)





