---
title: Android_M新特性
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_30.JPG-blog'
date: 2016-09-18 23:03:55
tags: [Android基础]
categories: [Android]
---

> Android5.0中添加了很多MD风格的控件，现在简单了解下各个控件的作用和使用效果
[Android M MD风格的介绍](http://blog.csdn.net/feiduclear_up/article/details/46514791)

<!--more-->

# Snackbar
> Snackbar提供了一个介于Toast和AlertDialog之间轻量级控件，它可以很方便的提供消息的提示和动作反馈。

```
Snackbar snackbar = Snackbar.make(inputLayout,"测试弹出提示",Snackbar.LENGTH_LONG);
                snackbar.show();
                snackbar.setAction("取消",new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        snackbar.dismiss();
                    }
                });
```

# TextInputLayout
> 具有MD风格的文本输入框父布局

```
搭配EditText使用，显示错误信息的提示
```
3. FloatingActionButton悬浮按键FAB继承者ImageView
4. TabLayout 标签布局搭配ViewPager使用
5. NavigationView导航布局实现抽屉效果搭配DrawerLayout使用

6. CoordinatorLayout 继承自FrameLayout
AppBarLayout 继承自LinerLayout

> 实现滑动的效果需要两种布局组合CoordinatorLayout 作为整体的父布局
> 1. AppBarLayout布局，作为滑动视图的父布局
> 2. 可滑动的VIew(RecyclerView)需要设置app:layout_behavior="@string/appbar_scrolling_view_behavior"属性
> 设置后该布局会自动放在AppBarLayout的下面

** 在AppBarLayout中放置你想滚动的视图，并设置相应的滚动属性，**
如果里面放在的是嵌套的布局文件的话，只对最外面的根局有效
属性有四种：
app:layout_scrollFlags=”scroll|enterAlways” 
 1. scroll: 所有想滚动出屏幕的view都需要设置这个flag- 没有设置这个flag的view将被固定在屏幕顶部。
 2. enterAlways: 这个flag让任意向下的滚动都会导致该view变为可见，启用快速“返回模式”。
 3. enterAlwaysCollapsed: 当你的视图已经设置minHeight属性又使用此标志时，你的视图只能已最小高度进入，只有当滚动视图到达顶部时才扩大到完整高度。
 4. exitUntilCollapsed: 滚动退出屏幕，最后折叠在顶端。

 加入可折叠的属性CollapsingToolbarLayout 
CollapsingToolbarLayout包裹 Toolbar 的时候提供一个可折叠的 Toolbar，
一般作为AppbarLayout的子视图使用。
如果CollapsingToolbarLayout 中不显示添加ToolBar
这应该在CollapsingToolbarLayout 中设置    android:minHeight="40dp"也可以达到折叠的效果




