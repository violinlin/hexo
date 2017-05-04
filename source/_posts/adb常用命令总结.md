---
title: adb常用命令总结
photos:
  - 'http://7xvvky.com1.z0.glb.clouddn.com/picture/picture21.jpg'
date: 2016-09-08 10:58:08
tags: [adb,Android]
---
> 总结常用的adb命令
<!--more-->
## adb服务的开启与关闭

```
//开启服务
adb start-server
//关闭服务
adb stop-server
```
## 查看当前的机器列表

```
adb devices
```
## 程序的安装

```
adb install [apk 所在目录]
```
## 查看设备WIFI的MAC地址

```
adb shell cat /sys/class/net/wlan0/address
```
## 查看文件
 1. 进入手机命令

```
adb shell
adb -s '设备信息' shell //指定机型操作
```
2. 切换到指定目录

```
cd data/data/包名/shared_prefs
ls 查看当前目录下的文件
```

3. 查看文件
 
```
cat 文件名称
```
## 获取屏幕的分辨率

```
adb shell wm size//屏幕分辨率
adb shell wm density//屏幕密度
```
## 查看安装的程序

```
adb shell pm list packages 
//添加过滤参数
adb shell pm list packages -s//获取系统应用
adb shell pm list packages -3//第三方应用
adb shell pm list packages '过滤字段'
```
##  向手机发送文件

```
adb push <要发送的文件路径>  <发送到的路径>
adb push test.text  sdcard/Movies
```
