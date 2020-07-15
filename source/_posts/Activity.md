---
title: Activity
photos:
  - '/img/pictures/picture1.jpg'
date: 2016-09-18 22:32:40
tags: [Android基础,Activity]
categories: [Android]
---

<!--more-->
# Activity 生命周期

![lifecycle](/img/activity_lifecycle.png)

## 多Activity启动生命周期

> Activity生命周期顺序执行如上，如果A启动B，在B界面再退回A两个Activity的生命周期会如何调用，下面简单验证下

1. 启动App,MainActivity创建生命周期如下

```java
D/Activity: MainActivity	onCreate
D/Activity: MainActivity	onStart
D/Activity: MainActivity	onResume
```
2. 从`MainActivity`启动`SecondActivity`

> 从日志可以看出`MainActivity`会先走`onPause()`方法，等`SecondActivity`界面走完创建流程，到`onResume()`之后，前一个界面才会走`onStop()`方法

```java
D/Activity: MainActivity	onPause
D/Activity: SecondActivity	onCreate
D/Activity: SecondActivity	onStart
D/Activity: SecondActivity	onResume
D/Activity: MainActivity	onSaveInstanceState
D/Activity: MainActivity	onStop
```
3. 从`SecondActivity`退回到`MainActivity`

> 当上一个界面`MainActivity`重新`onResume()`后，`SecondActivity`才会调`onStop(),onDestory()`方法

```java
D/Activity: SecondActivity	onPause
D/Activity: MainActivity	onReStart
D/Activity: MainActivity	onStart
D/Activity: MainActivity	onResume
D/Activity: SecondActivity	onStop
D/Activity: SecondActivity	onDestroy
```
> 从日志也可以看出无论从A启动到B,还是从B退回到A,当前显示的Activity会先走`onPause()`方法，目标Activity才会走创建或者重新回到前台流程。所以`onPause()`方法中不适合做一些耗时操作，会影响下一个界面的打开。

## 生命周期分类

### onCreate -> onDestory() 创建销毁期

> Activity 的创建和销毁回调

### 可见生存期 onStart() -> onStop()

> onStart()到onStop()之间，Activity对用户是可见的，当Activity调用onStop()后，说明该Activity位于后台用户不可见，这种状态极易被系统回收。

### 前台生存期 onResume() -> onPause()

> onResume() -> onPause() Activity属于前台运行状态，可以和用户进行交互

### onReStart()

> SecondActivity退出栈，MainActivity重新可见时，会调用onRestart() -> onStart() -> onResume()

### onSaveInstanceState()

> 在onStop()方法前调用，通过Bundle存数据，如果Activity被回收了，重新回退到该Activity时，可以在onCreate(Bundle bundle)中获取保存的数据

## QA
1. 在Activity中弹出一个Dialog，它的生命周期会怎么变化，如果启动一个Dialog主题的Activity呢？
> 弹出Dialog生命周期不会变化，Dialog依赖于Activity，Activity并不会走onPause()方法
> 启动一个Dialog主题的Activity，该Activity会走onPause()方法，但是因为没有完全被遮住，所以不会走onStop()方法

2. 在Activity中启动一个Dialog主题的Activity，onSaveInstanceState()会被调用嘛
> 会的，虽然onStop()方法后Activity才容易被回收，但是onSaveInstanceState()要早于onStop()调用，所以方法还是会被调用的，日志如下

```
D/MainActivity: onPause
D/DialogActivity: onCreate
D/DialogActivity: onStart
D/DialogActivity: onResume
D/MainActivity: onSaveInstanceState
```




# Activity 启动模式

![launch mode](/img/launchmode.png)

## onNewIntent方法的执行时机
1. 当Activity的启动模式为SingleTop
> 如果目标Activity位于栈顶，此时再次启动该Activity，不会创建新的Activity，该Activity的onNewIntent()方法会被调用
2. 当Activity的启动模式为SingleTask
> 如果栈中已经有目标Activity了，再次启动目标Activity，会先将目标Activity上的Activity全部出栈，然后调用目标Activity的onNewIntent()方法
3. 当Activity的启动模式为SingleInstance
> 如果已经创建过目标Activity，再次启动目标Activity，目标Activity会切换到前台，然后调用目标Activity的onNewIntent()方法，因为他是单独一个栈，所以不会使其他Activity出栈

```
D/MainActivity: onPause
D/MainActivity: onNewIntent
D/MainActivity: onResume
```

## Activity常见异常问题

## Activity 启动异常

> 通过隐式意图启动Activity时需要处理一些异常情况，例如目标Activity不存在、没有打开权限等。

## Permission Denial

> 打开目标Activity权限失败，例如目标Activity中设置了下面参数


 ```java
 android:exported="false"
 
 ```
具体错误信息 

```java
Permission Denial: starting Intent { act=android.intent.action.VIEW dat=plugin:// flg=0x10200000 cmp=cn.qbzsydsq.reader/demo.ad.dy.com.appdemo.plugin.PluginActivity } from ProcessRecord{80c0b66 26467:com.violin.violindemo/u0a931} (pid=26467, uid=10931) not exported from uid 11016
```

## No Activity found to handle Intent

> 手机上没有找到处理意图的目标Activity

具体错误信息 

```java
android.content.ActivityNotFoundException: No Activity found to handle Intent { act=android.intent.action.VIEW dat=plugi:// flg=0x10200000 }

```

 

