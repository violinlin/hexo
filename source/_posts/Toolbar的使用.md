---
title: Toolbar的使用
photos:
  - '/img/pictures/picture5.jpg'
date: 2017-04-19 20:44:39
tags: [Android,MD]
---

> `Toolbar`是在 Android 5.0 开始推出的一个 Material Design 风格的导航控件，用来代替以之前的 `ActionBar`。尽管Google诚意满满，使用`Toolbar`依然是一部踩坑填坑的血泪史。

<!--more-->

对于`Toolbar`Google给开发者留了很多可定制修改的余地。[官方文档](https://developer.android.google.cn/reference/android/support/v7/widget/Toolbar.html)
![](http://upload-images.jianshu.io/upload_images/2352140-4f074872909d1528.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 设置导航按键
* 设置程序Logo
* 设置主/副标题
* 设置一个或多个自定义控件
* 支持acition menu


## 1. Toolbar的引入

>  `Toolbar`用来替代`ActionBar`,所以Activity的`Theme`设置成`NoActionBar`的例如`Theme.AppCompat.Light.NoActionBar`或者在代码中去掉导航栏在`onCreate()`中添加`supportRequestWindowFeature(Window.FEATURE_NO_TITLE) `
 
1.1 添加兼容包依赖
 `Toolbar`是向下兼容的，所以使用时添加`support`依赖包` compile 'com.android.support:appcompat-v7:23.1.1'`
1.2  在布局文件中引入`Toolbar`

```xml
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize"
	    app:navigationIcon="@drawable/ic_drawer_home"//导航图标
		app:logo="@mipmap/ic_launcher"//程序logo
		app:title="Title"//主标题
		app:subtitle="SubTitle"//副标题>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="custom view"
            android:textColor="#fff"
            android:textSize="18sp" />

    </android.support.v7.widget.Toolbar>
```

1.3  在代码中设置`Toolbar`替代`ActionBar`

```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_toolbar);
        Toolbar toolbar= (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);//Toolbar 代替ActionBar

    }
```

程序预览
![](http://upload-images.jianshu.io/upload_images/2352140-d20d494edc2c726f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. 标题字体样式的自定义

> 简单的引入了`Toolbar`,但这并不能满足我们的开发要求。标题的字体样式我们总得按照设计稿来吧。

### 2.1 修改标题字体颜色

```
  app:titleTextColor="@color/colorAccent"//修改主标题字体颜色
  app:subtitleTextColor="@color/colorAccent"//修改副标题字体颜色
```

### 2.2 修改标题字体大小
修改字体大小就比较坑了，我没找到设置字体大小的直接属性。可能`Toolbar`的设计中会自动设置标题的字体大小为一个合适的值。其实用默认的大小基本ok。如果想要修改字体大小可以尝试下面的方法。

**我在样式文件中定义了需要样式的父类样式，所以后面 的样式都没写`parent`,他们都自动继承这个样式**
```
<!--Toolbar 样式-->
    <style name="Theme.Toolbar" parent="ThemeOverlay.AppCompat.ActionBar">
       
    </style>
```

2.2.1 在`styles.xml`定义标题的字体样式

```xml
  <!--Toolbar 主标题的字体和颜色-->
    <style name="Theme.Toolbar.Title">
        <item name="android:textSize">14sp</item>
        <item name="android:textColor">@color/colorAccent</item>
    </style>
    <!--Toolbar 副标题的字体和颜色-->
    <style name="Theme.Toolbar.SubTitle">
        <item name="android:textSize">12sp</item>
        <item name="android:textColor">@color/colorAccent</item>
    </style>
```

2.2.2 在布局文件中使用样式

```xml
  app:subtitleTextAppearance="@style/Theme.Toolbar.SubTitle"
  app:titleTextAppearance="@style/Theme.Toolbar.Title"
```
## 3. Action Menu的添加
由于上面已经通过`setSupportActionBar(toolbar)`将`Toolbar`设置成了程序的`ActionBar`。菜单布局的渲染和菜单按键的监听可以直接在`Activity`的回调方法中实现
3.1 创建菜单布局文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu1"
        android:icon="@drawable/ic_search"
        android:title="menu1"
        app:showAsAction="always" />
    <item
        android:id="@+id/menu2"
        android:icon="@drawable/ic_notifications"
        android:title="menu2"
        app:showAsAction="never" />
</menu>
```
3.2 加载菜单布局文件

```java
 @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.toobar, menu);
        return super.onCreateOptionsMenu(menu);
    }
```

3.3 设置菜单点击回调监听

```
 @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.menu1:
                Toast.makeText(ToolbarActivity.this, "item1:" + item.getTitle().toString(), Toast.LENGTH_SHORT).show();
                break;
            case R.id.menu2:
                Toast.makeText(ToolbarActivity.this, "item2" + item.getTitle().toString(), Toast.LENGTH_SHORT).show();
                break;

        }
        return super.onOptionsItemSelected(item);
    }
```
![](http://upload-images.jianshu.io/upload_images/2352140-9fcebbbe943b45f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 4. Action Menu样式的自定义

> 如图预览所示，这里会有下面几个问题：
1. 显示更多菜单按键由系统生成和其他图标不符
2. 菜单弹窗背景字体样式等的自定义
3. 菜单弹窗中子菜单项图标没有显示（运行到手机也没显示）

### 4.1 显示更多菜单图标的替换
4.1.1 在styles.xml样式文件中定义`Toolbar`的样式

```xml
  <!--Toolbar 样式-->
    <style name="Theme.Toolbar" parent="ThemeOverlay.AppCompat.ActionBar">
        <item name="actionOverflowButtonStyle">@style/Toolbar_action_menu</item>
    </style>
    <!--更多菜单图标-->
    <style name="Toolbar_action_menu">
        <item name="android:src">@drawable/ic_menu_more_overflow</item>
    </style>
```

4.1.2 在布局文件中为`Toolbar`指定样式文件

```xml
 android:theme="@style/Theme.Toolbar"
```

### 4.2 菜单弹窗背景、字体的自定义
4.2.1 在styles.xml样式文件中定义`Toolbar`菜单的样式

```
  <!--菜单上文字的大小和颜色-->
    <style name="Theme.Toolbar.PopMenu">
        <item name="android:textSize">18sp</item>
        <item name="android:textColor">@color/colorAccent</item>
        <item name="android:background">@color/cl_1B2137</item>
    </style>
```
4.1.2 在布局文件中为`Toolbar`弹窗菜单指定样式文件

```
  app:popupTheme="@style/Theme.Toolbar.PopMenu"
```

### 4.3 菜单弹窗子菜单图标不显示解决方案
4.3.1 添加菜单显示的方法

```java
//设置菜单选项图标是否显示
    private void setIconEnable(Menu menu, boolean enable) {
        if (menu != null) {
            if (menu.getClass().getSimpleName().equals("MenuBuilder")) {
                try {
                    Method m = menu.getClass().getDeclaredMethod("setOptionalIconsVisible", Boolean.TYPE);
                    m.setAccessible(true);
                    m.invoke(menu, enable);
                } catch (Exception e) {
                }
            }
        }
    }
```
4.3.2 在创建菜单的时候设置图标是否显示

```
 @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        setIconEnable(menu, true);
        getMenuInflater().inflate(R.menu.toobar, menu);
        return super.onCreateOptionsMenu(menu);
    }
```
在手机上运行显示截图如下
![](http://upload-images.jianshu.io/upload_images/2352140-6c4b816f22cae5b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/2352140-e9b65cd457e7d61f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 5. 关于主标题的显示问题
因为在代码中通过` setSupportActionBar(toolbar);`将`Toolbar`设置成了`ActionBar`,所以如果没有没置`Toolbar`的主标题，或者主标题设置为了空`app:title=""`，系统会获取`Activity`的`lable`字段设置为主标题。如果`Activity`的`lable`为空则会获取`application`的`label`字段设置`title`。解决这个问题需要在代码中添加`getSupportActionBar().setDisplayShowTitleEnabled(false);`