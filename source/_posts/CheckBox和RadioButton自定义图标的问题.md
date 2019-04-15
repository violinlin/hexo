---
title: CheckBox和RadioButton自定义图标的问题
photos:
  - '/img/pictures/picture9.jpg'
date: 2016-08-19 11:36:50
tags: [Android,bug]
---
> 在Android开发中CheckBox和RadioButton应该是最常用的控件了，但是在不同Android系统的手机上他们的显示样式又都不相同，为了软件的统一性开发中经常会自定义图标来解决这个问题。这里有两种自定义图标的方法：1.通过button属性；2.通过drawableLeft属性。但是在使用第一种方法时遇到了一点问题，这也是记录这篇博客的原因。
<!-- more-->
在开发过程中CheckBox和RadioButton遇到的问题是相同的，那么这里就主要以CheckBox为例来描述。这两个控件都是有两种状态的，所以使用了背景选择器来为它们的不同状态配置不同的图片。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/checkbox_checked" android:state_checked="true">

    </item>

    <item android:drawable="@drawable/checkbox_unchecked" android:state_checked="false"></item>

    <item android:drawable="@drawable/checkbox_unchecked">

    </item>
</selector>
```

## 1.使用`button`属性自定义CheckBox的图标

通过`button` 属性为CheckBox重新定义图标，同时可以通过`paddingLeft`属性设置图标和文本之间的间隔。

``` xml
 <CheckBox
        android:button="@drawable/check_selector"
        android:checked="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="不在显示"
        android:paddingLeft="5dp"
        />
```

这种方法看上去还不错，万万没想到碰到一个诡异的bug,问题是这样的
左边是异常情况，图片遮住了文字。右边是正常显示情况。
![](http://upload-images.jianshu.io/upload_images/2352140-d3dd97da9e7f4d8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
刚开始以为是屏幕适配的问题，也按着这个思路去解决了。但是后来发现在4.4及以上的系统上没有问题（没那么多系统的手机，4.4及以上没发现问题,唯一有的低版本4.0和4.2都有这个bug）。要是通过`paddingLeft`属性处理的话在高版本的系统上又会显得间距较大。
后来又惊奇的发现如果继承的是`Activity`的话在低版本的手机上会出现同样的问题，如果继承的是v7包里的`AppCompatActivity`则问题没有再出现。**是代码的bug么，问题应该是paddingLeft计算的起始位置不同，跪求解答啊**。无奈由于一些原因我们代码只能继承Activity，so使用另一种方法吧。

## 2.通过`drawableLeft`属性自定义CheckBox的图标

CheckBox和RadioButton都是继承自TextView，可以使用TextView的`drawableLeft`属性实现效果，同时可以使用`drawablePadding`设置图标和文本之间的距离。**使用drawable属性需要将button和background的值设置为null**

``` xml
 <CheckBox
        android:button="@null"
        android:background="@null"
        android:drawableLeft="@drawable/check_selector"
        android:drawablePadding="5dp"
        android:checked="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="不在显示"
        />
```