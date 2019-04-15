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
## 指定目标设备
```
adb -s 'serialNumber' 
adb -d 指定当前唯一通过USB连接的Android设备
adb -e 指定当前唯一运行的虚拟设备
```

## 进入手机

```
adb shell

```
## 切换到指定目录

```
cd data/data/包名/shared_prefs
ls 查看当前目录下的文件
```

## 查看相应包名的清单信息
```
adb shell dumpsys package[pkname]
```
## 安装应用
```
adb install [-lrtsdg] [apk 路径]
```
| 参数 | 作用 |
|--------|--------|
|   -l     |  将应用安装到保护目录/mnt/asec      |
|   -r     |  允许覆盖安装      |
|   -t     |  允许安装 AndroidManifest.xml 里 application 指定 android:testOnly="true" 的应用      |
|   -s     |  将应用安装到sdcard      |
|   -d     |  允许覆盖降级安装     |
|   -g     |  授予所有运行时权限     |

如果包的签名咩有问题，在部分机器（如Lenovo K920）上扔会遇到`[INSTALL_FAILED_VERIFICATION_FAILURE]`安装失败的情况。解决方法如下：
1. 将安装包复制到手机安装目录 `adb push [xx.apk] /data/local/tmp/xx`
2. 安装apk `adb shell pm install /data/local/tmp/xx`




## 卸载应用
```
adb uninstall [pkname]
```
## 将设备文件拖到本地
```
adb pull [手机文件目录] [电脑文件目录]
```
## 将本地文件拖到设备
```
adb push [电脑文件路径] [手机文件目录]
```
## 清空应用缓存
```
adb shell pm clear  [pkname]
```
## 查看debug包的沙盒数据
```
adb shell
run-as [pkname]
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
##  传输文件

```
adb push <要发送的文件路径>  <发送到的路径>
adb push test.text  sdcard/Movies
```
## 屏幕截图

```

// 截屏并保存到电脑的当前目录
adb exec-out screencap -p > picture_name.png

// 截屏并保存到当前设备
adb shell screencap -p /sdcard/sc.png 

```
## 模拟屏幕滑动

```
adb shell input swipe [startX、startY、endX、endY、durationTime(ms)]
```
