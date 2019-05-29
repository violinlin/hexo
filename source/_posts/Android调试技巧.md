---
title: Android进入调试模式的三种技巧
photos:
  - /img/pictures/picture19.jpg
date: 2018-05-29 17:36:22
tags: [adb,Android,AndroidStudio]
categories:
---
> Android开发过程中难免会遇到各种问题，通常我们会通过打印Log日志或者Debug模式来分析问题。这里介绍下Android程序进入到Debug的多种方式，可以针对不同场景使用。
ps:当然只限于debug包，正式包进不了调试模式。[官方文档](https://developer.android.com/studio/debug)


<!--more-->

# 1. 直接发布Debug包进入调试模式

> 首先可以直接通过 `Debug app` 按键发布包，安装成功后即进入调试模式。这种方式的问题就是每次进入调试模式都需要发布包，效率比较低。

![](/img/debug_run.png)

# 2. `Attach debugger to Android process` 方式进入调试模式

> 启动手机上安装好的待调试的debug包，在AS的工具栏上点击`Attach debugger to Android process`按键，选择待调试的进程即进入调试模式。这种方式较第一种效率比较高，但是attach时需要保证进程已经启动，
不方便调试`Application`中的一些初始化代码。

![](/img/debug_attach.png)

# 3. 将应用设置为待调试模式

> 这种方式是将指定应用标志为调试模式，每次启动应用都会弹出`Waiting For Debugger`的弹窗。然后可以通过第二种方式`Attach debugger to Android process`连接应用进程进入调试模式。
这种方式可以解决第二种方式中`Application`的一些初始化代码不方便调试的问题，因为调试弹窗弹起的时候进程已经启动。有两种方式可以将目标应用标志为调试模式。

## 1. 通过adb命令`set-debug-app`方式标志

```
adb shell am set-debug-app -w --persistent <package_name>

//-w: 让程序等待被attach
//--persistent: 让程序每次启动都等待被attach

```
添加了`persistent` 参数后每次启动app都会弹出调试弹窗，完成调试后需要通过下面命令移除标志

```
adb shell am clear-debug-app <package_name>
```

## 2. 通过手机设置进行标志

> 在手机开发者选项中选择要调试的应用，勾选等待调试器（完成调试后需要关掉调试器，不然每次启动都会弹出调试弹窗）。如下图

![](/img/debug_device.png)

通过上面任意一种方式将app标志为调试模式后，启动应用会弹出调试弹窗.然后通过第二种方式attach到应用进程就可以进入调试模式了

![](/img/debug_wait.png)

# 总结

> 综上三种方式，开发中经常使用的应该是第二种，如果需要调试进程初始化部分的代码可以使用第三种方式。第一种方式因为每次都要打包发布效率较低，不建议使用
，或者只在第一次发布程序的时候使用。


