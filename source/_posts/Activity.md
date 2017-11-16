---
title: Activity
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_19.JPG-blog'
date: 2016-09-18 22:32:40
tags: [Activity]
categories: [Android]
---

<!--more-->

# Activity 生命周期

![lifecycle](http://7xvvky.com1.z0.glb.clouddn.com/blog/activity/activity_lifecycle.png)

# Activity 启动模式

![launch mode](http://7xvvky.com1.z0.glb.clouddn.com/blog/activity/launchmode.png)

# onNewIntent方法的执行时机

```
当此Activity的实例已经存在，并且此时的启动模式为SingleTask和SingleInstance，另外当这个实例位于栈顶且启动模式为SingleTop时也会触发onNewInstent()。
```
