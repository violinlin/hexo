---
title: Activity启动模式
photos:
  - '/img/pictures/picture2.jpg'
date: 2017-11-15 17:42:14
tags: [Activity]
categories: [Android]
---

# Activity启动模式

<!--more-->

## 清单文件中配置启动模式`launchMode`
[官方文档](https://developer.android.google.cn/guide/topics/manifest/activity-element.html#lmode)

![launch mode](/img/launchmode.png)

### singleTask

> 设置A Activity的启动模式为`singleTask`,启动顺序为`A-->B-->C-->A`各Activity的生命周期如下: 按照启动顺序，各个界面依次调用`onCreate()`方法。当从`C`启动到`A`时，`B`界面走了`onDestory()`方法，`A`界面不会重新创建，走了`onNewIntent()`方法，随后`C`界面销毁走`onDestory()`方法

**当Activity的启动模式设置为`singleTask`时，任务栈中只会创建一次这个Activity的对象，当重复启动该Activity时会调用`onNewIntent()`方法，而不是`onCreate()`方法，同时会将任务栈中该Activity上面的Activity清除出栈。**

```java
D/whl: AActivity------onCreate
D/whl: BActivity------onCreate
D/whl: CActivity------onCreate
D/whl: BActivity------onDestroy
D/whl: AActivity------onNewIntent
D/whl: CActivity------onDestroy 
```
### singleTop

> 设置B Activity的启动模式为`singleTop`,启动顺序为`A-->B-->B`各Activity的生命周期如下: 从`A`启动到`B`，这时`BActivity`位于栈顶，再次跳转`BActivity`时，界面不会重新创建，走`onNewIntent()`方法

**当Activity的启动模式设置为`singleTop`时，任务栈中允许创建多个该Activity的对象。但是，如果当该Activity位于任务栈顶时再次启动该界面，不会创建新的对象。**

```java
D/whl: AActivity------onCreate
D/whl: BActivity------onCreate
// BActivity 已经在栈顶，再次启动BActivity
D/whl: BActivity------onPause
D/whl: BActivity------onNewIntent
D/whl: BActivity------onResume
```


## Intent中配置FLAG
> 关于Activity的启动模式除了在清单中静态配置`launchMode`之外，也可以在代码中通过设置Intent的FLAG动态设置。[官方文档](https://developer.android.google.cn/reference/android/content/Intent.html#setFlags%28int%29)

### FLAG_ACTIVITY_SINGLE_TOP
> 作用同`launchMode`设置`singleTop`

### FLAG_ACTIVITY_CLEAR_TOP
> If set, and the activity being launched is already running in the current task, then instead of launching a new instance of that activity, all of the other activities on top of it will be closed and this Intent will be delivered to the (now on top) old activity as a new Intent.

> 按照官方文档的解释，当设置这个FLAG通过Intent启动目标Activity时，在栈中位于目标Activity之上的界面都会被清除出栈。目标Activity会位于栈顶，接受新的Intent。根据文档，我的理解是目标Activity的`onNewIntent()`方法会被调用，且不会创建新的对象。通过Demo来验证一下`A-->B-->C-->B`。依次启动A、B、C界面，当C启动B时，设置`FLAG_ACTIVITY_CLEAR_TOP`FLAG。

```java
AActivity------onCreate
BActivity------onCreate
CActivity------onCreate
BActivity------onDestroy
BActivity------onCreate
CActivity------onDestroy
```
**运行结果好像和文档描述不太一样，结论是设置`FLAG_ACTIVITY_CLEAR_TOP`FLAG启动目标Activity，会将栈中位于目标Activity上面的Activity清理出栈，和launchMode设置singleTask不同，这个FLAG同时导致目标Activity也会销毁然后重建**

*Ps. 如果想要清除目标Activity上面的Activity，同时目标Activity不会重建可以设置FLAG`intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP|Intent.FLAG_ACTIVITY_SINGLE_TOP)`*

### FLAG_ACTIVITY_CLEAR_TASK
> If set in an Intent passed to Context.startActivity(), this flag will cause any existing task that would be associated with the activity to be cleared before the activity is started. That is, the activity becomes the new root of an otherwise empty task, and any old activities are finished. This can only be used in conjunction with FLAG_ACTIVITY_NEW_TASK.

> 设置这个FLAG会清空Activity栈，但是使用会有问题，它需要搭配另外一个FLAG联合使用`intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK|Intent.FLAG_ACTIVITY_CLEAR_TASK)`
A-->B-->C 依次启动A、B、C当重B启动C时添加FLAG


```java
AActivity------onCreate
BActivity------onCreate
AActivity------onDestroy
CActivity------onCreate
BActivity------onDestroy
```
**当设置`Intent.FLAG_ACTIVITY_NEW_TASK|Intent.FLAG_ACTIVITY_CLEAR_TASK`时，会清空当前任务栈，目标Activity重新创建，且位于新栈的最底部**

### FLAG_ACTIVITY_REORDER_TO_FRONT

> If set in an Intent passed to Context.startActivity(), this flag will cause the launched activity to be brought to the front of its task's history stack if it is already running.
这个FLAG理解比较简单A-->B-->C-->B 依次启动A、B、C，栈中的顺序为ABC,当从C启动B时设置`FLAG_ACTIVITY_REORDER_TO_FRONT`日志如下：

```java
AActivity------onCreate
BActivity------onCreate
CActivity------onCreate
BActivity------onNewIntent
```
**`B`界面会重新回到栈顶，`C`界面也没有出栈，栈中顺序改为了A-->C-->B。但是使用中存在一个问题，通过设置这个标签启动`B`到栈顶，如果按返回键日志显示`B`出栈`A`和`C`并没有出栈但是程序会退回到桌面，重新进入程序栈中顺序为A-->C**


## 目前就研究了这几个，有兴趣再研究其他的吧