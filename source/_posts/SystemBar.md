---
title: SystemBar
photos:
  - '/img/pictures/picture4.jpg'
date: 2017-04-25 14:32:18
tags: [Android,MD]
---

> SystemBars 主要包括两个状态栏`StatusBar`和导航栏`NavigationBar`。状态栏用来显示时间、电量等系统信息，导航栏一般在无实体按键的手机上，用来代替回退、Home、和菜单键。

<!--more-->

## 1. 沉浸模式的应用
[沉浸模式](https://developer.android.com/training/system-ui/immersive.html#sticky)
Android 4.4(API 19) 以上支持了沉浸模式`IMMERSIVE MODE` 可以让布局在整个手机屏幕上显示，隐藏掉系统栏。一般游戏、图片预览、电影等app都会使用这种模式。这里做一个图片预览的简单应用。

### 1.1 在主题文件中设置系统栏透明

```xml
 <item name="android:windowTranslucentStatus">true</item><!--设置状态栏透明-->
                <item name="android:windowTranslucentNavigation">true</item><!--设置导航了透明-->
```

### 1.2 隐藏系统栏
> 默认界面获取焦点的时候隐藏系统栏，所以在`onResume()`方法中进行下面操作

```java
 @Override
    protected void onResume() {
        super.onResume();
        hideSysBar();
    }

    private void hideSysBar() {
        getWindow().getDecorView().setSystemUiVisibility(
                HIDE_SYSBAR_FLAG);
        relativeLayout.startAnimation(mHiddenAction);
        relativeLayout.setVisibility(View.GONE);

    }
```
### 1.3 显示系统栏
> 点击界面时显示系统通知栏

```java
 private void showSysBar() {
        decorView.setSystemUiVisibility(SHOW_SYSBAR_FLAG);
        relativeLayout.startAnimation(mShowAction);
        relativeLayout.setVisibility(View.VISIBLE);
    }
```
### 2. 补充说明
2.1 为了保证隐藏掉系统栏后布局能够自适应屏幕，需要在布局文件中添加` android:fitsSystemWindows="true"`
 
 2.2 系统栏FLAG说明
 

```java
  private final int HIDE_SYSBAR_FLAG = View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION // hide nav bar
            | View.SYSTEM_UI_FLAG_FULLSCREEN // hide status bar
            | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY;//设置沉浸模式,系统栏自动隐藏

    private final int SHOW_SYSBAR_FLAG = View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN;//显示系统栏flag,同时是布局在系统栏底部显示
```

2.3 视图动画说明

```java
 TranslateAnimation mShowAction;

    private void showAction() {
        mShowAction = new TranslateAnimation(Animation.RELATIVE_TO_SELF, 0.0f,
                Animation.RELATIVE_TO_SELF, 0.0f, Animation.RELATIVE_TO_SELF,
                -1.0f, Animation.RELATIVE_TO_SELF, 0.0f);
        mShowAction.setDuration(500);
    }

    TranslateAnimation mHiddenAction;

    private void hideAction() {

        mHiddenAction = new TranslateAnimation(Animation.RELATIVE_TO_SELF,
                0.0f, Animation.RELATIVE_TO_SELF, 0.0f,
                Animation.RELATIVE_TO_SELF, 0.0f, Animation.RELATIVE_TO_SELF,
                -1.0f);
        mHiddenAction.setDuration(500);
    }
```

![效果预览](/img/blog_sysbar.gif)



