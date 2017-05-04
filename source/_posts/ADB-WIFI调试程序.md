---
title: ADB--WIFI调试程序
date: 2016-07-16 20:42:54
tags: [Android,adb]
photos:
- http://7xvvky.com1.z0.glb.clouddn.com/picture/picture11.jpg
---
# 关于ADB
在开发调试Android程序时我们需要通过adb工具在我们的手机和电脑之间建立连接，通常情况我们都是使用数据线，其实adb还提供了另外一种方式通过**tcpip**建立连接。这里给大家介绍两种wifi连接电脑的方法，其中第一种方法手机不用root权限。

<!-- more -->
## adb简介
> Android Debug Bridge (adb)安卓调试桥，用来管理模拟器或设备。它采用的是C/S模式，主要包括三个部分：
> - **A client** 客户端Client运行在自己的电脑上，可以通过adb命令 `adb start-server`启动Client，也可以通过ADT或者DDMS创建Client。
> - **A daemon** Daemon作为后台程序运行在手机或者模拟器上。
> - **server** Server最为后台程序运行在自己的电脑上,用来管理Client和Daemon之间的信息交互。

## adb端口问题
- Server端启动绑定的是本机的5037端口。Client端用5037与服务器端对话。
- Deamon都会取5555到5585之间两个连续的端口，其中奇数端口是负责与adb链接，偶数端口是负责与控制台链接。服务器端通过扫描5555到5585之间的奇数端口来寻找模拟器或设备实例并与找到的建立链接。 　

# 通过wifi调试程序

**注意** 使用wifi调试程序首先确保你的电脑和手机在同一个wifi环境下。同时后面会用到一些adb的命令，所以先给你的电脑配置adb的环境变量，配置完成后我们可以直接在AndroidStudio的Terminal中敲adb命令了。具体的配置方法这里就不做介绍了，你那么聪明肯定会配置的。

## 无需手机root权限的配置方法

这种方法不需要手机有root权限，但是在第一次连接时需要数据线连接电脑，配置好之后数据线则可以断开。

使用命令`adb devices`查看手机是否连接成功

![](http://upload-images.jianshu.io/upload_images/2352140-efc18e9904edf47e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用命令`adb tcpip [port]`让手机的某个端口处于监听状态
端口后的范围为5555-5585的奇数端口。默认从5555开始，大家也可以和我一样配置该端口。

![](http://upload-images.jianshu.io/upload_images/2352140-0e137c4c7c235c9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**返回`restarting in TCP mode port : 5555`代表端口已经处于了监听状态。这个时候就可以断开手机连接的数据线了**。

在手机的wifi设置中查看你的ip地址**[ip-address]**,使用命令行`adb connect [ip-address]：[port-num] `连接手机,adb connect 手机的ip地址：上面配置的端口号。
![](http://upload-images.jianshu.io/upload_images/2352140-454c5b380c179614.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

返回`connected to [ip-address]：[port-num]`表示成功连接了手机，现在可以通过wifi在发布调试程序了。再次通过`adb-devices`查看连接设备的列表
![](http://upload-images.jianshu.io/upload_images/2352140-b9170c4e2b8f9de3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


----------


如果觉得敲命令行太麻烦也可以下载AndroidStudio的插件**Android WiFi ADB**
![](http://upload-images.jianshu.io/upload_images/2352140-4b342d21c1699203.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过数据线连接电脑，在插件显示的devices列表中选择连接的设备，点击**connect**按键，提示成功后拔掉数据线。
![](http://upload-images.jianshu.io/upload_images/2352140-9206c6c18e639eb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 需要手机root权限的配置方法

> 上面所讲的方法在第一次连接时都需要数据线的连接，如果手边没有数据线就不能连接电脑了吗？当然不是，我们回顾一下，上面的方法中我们使用数据线的目的是执行`adb tcpip [port]`命令，如果手机自己执行这个命令不就行了！方法是可行的不过执行这个命令得获取到手机的root权限。

这里给大家推荐一款软件**[WirelessADB](http://www.wandoujia.com/apps/me.meowo.adb)**方便连接，**手机得获取root权限，不然软件无法运行**。软件安装运行成功后直接根据界面的提示在AndroidStudio的Terminal中执行connect命令进行链接。
![](http://upload-images.jianshu.io/upload_images/2352140-eb916c7a7fbd21c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 断开wifi连接

停止wifi调试的时候可以通过`adb disconnect [ip-address]：[port-num] `来中断连接。

![](http://upload-images.jianshu.io/upload_images/2352140-3ba4fbd6886d691e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)