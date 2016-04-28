---
title: 2013-03-23-用Intent切换Activity
date: 2013-03-23 09:38:48
tags:
categories: Android
---

An intent is an abstract description of an operation to be performed. It can be used with startActivity to launch an Activity, broadcastIntent to send it to any interested BroadcastReceiver components, and startService(Intent) or bindService(Intent, ServiceConnection, int) to communicate with a background Service.
Intent是对一个要执行的操作的抽象描述。通过``Intent``，你可以使用``startActivity()``来启动一个``Activity``；使用``sendBroadcast()、sendOrderedBroadcast()、sendStickyBoradcast（）``等方法将其发送给任何感兴趣的``BroadcastReceiver``组件；或者使用``startService(Intent)``和``bindService(Intent, ServiceConnection, int)``来和一个后台服务通信。

An Intent provides a facility for performing late runtime binding between the code in different applications. Its most significant use is in the launching of activities, where it can be thought of as the glue between activities. It is basically a passive data structure holding an abstract description of an action to be performed.

``Intent``提供了在不同``Applications``间实施延时绑定的方法。它主要用来切换不同的``Activity``，可以看作是``Activities``间的通信媒介。
常见的用法有:
```
startActivity()           //启动另外一个Activity
startActivityForResult()    //启动一个Activity并监听其返回值

startService()            //启动一个服务
bindService()             //绑定一个后台服务

sendBroadcast()           //发送一个广播
sendOrderedBroadcast()    //发送顺序广播
sendStickyBoradcast()      //发送滞留广播，消息不会丢失
```

一个``Android``程序可能有多个``Activity``，或者可以说有多个界面。最常见的就是很多程序在启动时都会有个全屏的启动画面，延时几秒后进入主界面，其实现方法（之一）就是创建两个``Activity``，启动时进入启动画面的``Activity``，几秒种后自动跳转到主界面的``Activity``。

AndroidManifest.xml:
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.ljgabc.switchactivity"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="18" />

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >

        <!-- 每个Activity都要在这里注册，不然程序运行时会由于找不到Activity而报错 -->
        <activity
            android:name="com.ljgabc.switchactivity.SplashActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <!-- 这里标识程序启动时进入的那个Activity，有且只有一个 -->
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- 启动界面完成后跳转到此Activity -->
        <activity android:name="com.ljgabc.switchactivity.MainActivity"></activity>
    </application>
</manifest>
```

SplashActivity.java:
```
package com.ljgabc.switchactivity;

import android.os.Bundle;
import android.os.Handler;
import android.app.Activity;
import android.content.Intent;
import android.view.Window;
import android.view.WindowManager;

/*
 * 启动界面的Activity，显示LOGO两秒后跳转到MainActivity，并销毁自己
 */
public class SplashActivity extends Activity {

    final private int SPLASH_TIME = 2000;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        /*
         * 设置全屏
         */
        getWindow().setFlags(WindowManager.LayoutParams. FLAG_FULLSCREEN ,      
                WindowManager.LayoutParams. FLAG_FULLSCREEN);
        
        /*
         * 设置隐藏标题栏
         */
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        
        /*
         * 2秒后跳转到主界面
         */
        new Handler().postDelayed(new Runnable() {
            public void run() {
                launchMainActivity();
            }
        }, SPLASH_TIME);
        
        setContentView(R.layout.activity_splash);
    }   
    
    /*
     * 利用Intent切换到主Activity
     */
    private void launchMainActivity() {
        /*
         * 创建一个intent，从当前Activity指向要跳转的Activity
         */
        Intent intent = new Intent(this, MainActivity.class);
        
        /*
         * 启动目标Activity
         */
        startActivity(intent);
        
        /*
         * 启动画面只需要程序开始时显示一次，显示完后即可退出
         */
        finish();
    }
}
```

MainActivity.java
```
package com.ljgabc.switchactivity;

import android.os.Bundle;
import android.view.Window;
import android.app.Activity;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        /*
         * 设置隐藏标题栏
         */
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        
        setContentView(R.layout.activity_main);
    }   
}
```

效果：

{<1>}![demo](http://ljgabc.tk/img/switchactivity.gif)
