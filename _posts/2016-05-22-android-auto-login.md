---
layout: post
title: Android之记住密码与自动登陆实现
date: 2016-5-22
categories: blog
tags: [android]
description: Android之记住密码与自动登陆实现
---   
### 本文主要讲述了利用sharedpreference实现记住密码与自动登陆功能  

1. 根据checkbox的状态存储用户名与密码
2. 将结果保存在自定义的application中，成为全局变量


### 布局文件  

```
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:fitsSystemWindows="true">

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingTop="56dp"
        android:paddingLeft="24dp"
        android:paddingRight="24dp">

        <ImageView android:src="@drawable/logo"
            android:layout_width="wrap_content"
            android:layout_height="72dp"
            android:layout_marginBottom="24dp"
            android:layout_gravity="center_horizontal" />

        <!--  Email Label -->
        <android.support.design.widget.TextInputLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp">
            <EditText android:id="@+id/input_email"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="textEmailAddress"
                android:hint="Phone" />
        </android.support.design.widget.TextInputLayout>

        <!--  Password Label -->
        <android.support.design.widget.TextInputLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="8dp">
            <EditText android:id="@+id/input_password"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="textPassword"
                android:hint="Password"/>
        </android.support.design.widget.TextInputLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_horizontal"
            >
            <CheckBox
                android:id="@+id/rm_pass"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="记住密码"
                android:layout_marginRight="30dp"
                android:checked="true"
                />
            <CheckBox
                android:id="@+id/au_login"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="自动登陆"
                android:layout_marginLeft="30dp"
                />
        </LinearLayout>

        <android.support.v7.widget.AppCompatButton
            android:id="@+id/btn_login"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:layout_marginBottom="24dp"
            android:padding="12dp"
            android:text="Login"/>

        <TextView android:id="@+id/link_signup"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="24dp"
            android:text="No account yet? Create one"
            android:gravity="center"
            android:textSize="16dip"/>

    </LinearLayout>
</ScrollView>
```


### 登陆界面

```
package com.zj.login;

import android.app.Activity;
import android.app.ProgressDialog;
import android.content.Intent;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.util.Patterns;
import android.view.View;
import android.widget.Button;
import android.widget.CheckBox;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import com.zj.cafetriafood.R;

import butterknife.Bind;
import butterknife.ButterKnife;

/**
 * A login screen that offers login via email/password.
 */
public class LoginActivity extends AppCompatActivity  {
    private static final String TAG = "LoginActivity";
    private static final int REQUEST_SIGNUP = 0;
    private SharedPreferences sp;

    @Bind(R.id.input_email) EditText _emailText;
    @Bind(R.id.input_password) EditText _passwordText;
    @Bind(R.id.btn_login) Button _loginButton;
    @Bind(R.id.link_signup) TextView _signupLink;
    @Bind(R.id.rm_pass) CheckBox _rmpass;
    @Bind(R.id.au_login) CheckBox _aulogin;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        ButterKnife.bind(this);
        sp = this.getSharedPreferences("userInfo", Activity.MODE_PRIVATE);
        _loginButton.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                login();
            }
        });

        _signupLink.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
                // Start the Signup activity
                Intent intent = new Intent(getApplicationContext(), SignupActivity.class);
                startActivityForResult(intent, REQUEST_SIGNUP);
            }
        });



        if(sp.getBoolean("ISCHECK", false))
        {
            _rmpass.setChecked(true);

            _emailText.setText(sp.getString("USER_NAME", ""));
            _passwordText.setText(sp.getString("PASSWORD", ""));

            if(sp.getBoolean("AUTO_ISCHECK", false))
            {
                //设置默认是自动登录状态
                _aulogin.setChecked(true);
                //跳转界面
                //Intent intent = new Intent(LoginActivity.this,MainActivity.class);
                //startActivity(intent);
                this.finish();

            }
        }

    }

    public void login() {
        Log.d(TAG, "Login");

        if (!validate()) {
            onLoginFailed();
            return;
        }

        _loginButton.setEnabled(false);

        final ProgressDialog progressDialog = new ProgressDialog(LoginActivity.this,
                R.style.AppTheme_Dark_Dialog);
        progressDialog.setIndeterminate(true);
        progressDialog.setMessage("Authenticating...");
        progressDialog.show();

        String email = _emailText.getText().toString();
        String password = _passwordText.getText().toString();

        // TODO: Implement your own authentication logic here.
        Log.i("test","email+password="+email+","+password);


        if(!email.equals("123")||!password.equals("123456"))
        {
            progressDialog.dismiss();
            _loginButton.setEnabled(true);
            _emailText.setText("");
            _passwordText.setText("");
            Toast.makeText(getApplication(), "用户名或密码错误", Toast.LENGTH_SHORT).show();
            return;
        }

        if(_rmpass.isChecked())
        {
            //记住用户名、密码、
            SharedPreferences.Editor editor = sp.edit();
            editor.putString("USER_NAME", email);
            editor.putString("PASSWORD", password);


            editor.commit();

            sp.edit().putBoolean("ISCHECK", true).commit();


        }else
        {
            sp.edit().putBoolean("ISCHECK", true).commit();
        }




        if (_aulogin.isChecked())
        {
            sp.edit().putBoolean("AUTO_ISCHECK", true).commit();
        }else
        {
            sp.edit().putBoolean("AUTO_ISCHECK", false).commit();
        }


            new android.os.Handler().postDelayed(
                new Runnable() {
                    public void run() {
                        // On complete call either onLoginSuccess or onLoginFailed
                        onLoginSuccess();
                        // onLoginFailed();
                        progressDialog.dismiss();
                    }
                }, 3000);
    }






    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == REQUEST_SIGNUP) {
            if (resultCode == RESULT_OK) {

                // TODO: Implement successful signup logic here
                // By default we just finish the Activity and log them in automatically
                this.finish();
            }
        }
    }

    @Override
    public void onBackPressed() {
        // Disable going back to the MainActivity
        moveTaskToBack(true);
    }

    public void onLoginSuccess() {
        _loginButton.setEnabled(true);
        finish();
    }

    public void onLoginFailed() {
        Toast.makeText(getBaseContext(), "Login failed", Toast.LENGTH_LONG).show();

        _loginButton.setEnabled(true);
    }

    public boolean validate() {
        boolean valid = true;

        String email = _emailText.getText().toString();
        String password = _passwordText.getText().toString();

        if (email.isEmpty() || !Patterns.PHONE.matcher(email).matches()) {
            _emailText.setError("enter a valid phone number");
            valid = false;
        } else {
            _emailText.setError(null);
        }

        if (password.isEmpty() || password.length() < 4 || password.length() > 10) {
            _passwordText.setError("between 4 and 10 alphanumeric characters");
            valid = false;
        } else {
            _passwordText.setError(null);
        }

        return valid;
    }
}

```


### MainActivity

```
package com.zj.cafetriafood;

import android.app.Activity;
import android.content.Intent;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.widget.TextView;

import com.zj.application.MyApplication;
import com.zj.login.LoginActivity;

import butterknife.Bind;
import butterknife.ButterKnife;


public class MainActivity extends AppCompatActivity {

    private SharedPreferences sp;
    private MyApplication myApplication;

    @Bind(R.id.text_user) TextView text_user;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        sp = this.getSharedPreferences("userInfo", Activity.MODE_PRIVATE);

        if(sp.getBoolean("AUTO_ISCHECK", false))
        {
            myApplication= (MyApplication) getApplication();
            myApplication.setUsername(sp.getString("USER_NAME",""));
        }else
        {
            Intent intent = new Intent(this, LoginActivity.class);
            startActivity(intent);
        }

        text_user.setText(myApplication.getUsername());


    }



}
```


### MyApplication

```
package com.zj.application;

import android.app.Application;

/**
 * Created by jjx on 2016/5/22.
 */
public class MyApplication extends Application{

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    String username;

    @Override
    public void onCreate() {
        super.onCreate();
        setUsername("用户名");
    }
}
```

### 参考链接

[Android 记住密码和自动登录界面的实现（SharedPreferences 的用法） - liuyiming_的专栏 - 博客频道 - CSDN.NET](http://blog.csdn.net/liuyiming_/article/details/7704923)


[Android中Application类用法 - Harvey Ren - 博客园](http://www.cnblogs.com/renqingping/archive/2012/10/24/Application.html)

Application对象的生命周期是整个程序中最长的，它的生命周期就等于这个程序的生命周期。因为它是全局的单例的，所以在不同的Activity,Service中获得的对象都是同一个对象。所以可以通过Application来进行一些，如：数据传递、数据共享和数据缓存等操作。

在Android中，可以通过继承Application类来实现应用程序级的全局变量，这种全局变量方法相对静态类更有保障，直到应用的所有Activity全部被destory掉之后才会被释放掉。


### 效果如下 


![这里写图片描述](http://img.blog.csdn.net/20160522164451557)




















