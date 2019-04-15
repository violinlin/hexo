---
title: Apk的安装卸载等操作介绍
photos:
   - '/img/pictures/picture1.jpg'
date: 2016-09-19 13:45:01
tags: [Android]
---
通过包名实现程序的打开、卸载、更新等操作，apk文件的安装等

<!--more-->

## 监听程序的安装和卸载
在清单文件中注册广播，添加过滤器

```xml
<receiver
            android:name=".kit.packManager.PackReceiver"
            android:label="PackReceiver">
            <intent-filter>
                <action android:name="android.intent.action.PACKAGE_ADDED" />
                <action android:name="android.intent.action.PACKAGE_REPLACED" />
                <action android:name="android.intent.action.PACKAGE_REMOVED" />

                <data android:scheme="package" />
            </intent-filter>
        </receiver>
```
在接收到广播时进行相应的操作

```java
//获取发送广播的包名
 String packName = intent.getData().getSchemeSpecificPart();
 
```
通过Action判断当前的包操作

| Action      |     操作 |   
| :-------- | --------:|
| Intent.ACTION_PACKAGE_ADDED| 安装 |
| Intent.ACTION_PACKAGE_REPLACED | 更新 |
| Intent.ACTION_PACKAGE_REMOVED | 卸载 |

## 通过包名打开程序

```java
 Intent intent = context.getPackageManager().getLaunchIntentForPackage(packName);
        if (intent != null) {
            context.startActivity(intent);
        }
```

## 通过apk目录安装程序

```java
  Intent intent = new Intent(Intent.ACTION_VIEW);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setDataAndType(Uri.parse("file://" + packPath), "application/vnd.android.package-archive");
        context.startActivity(intent);
```
## 通过包名卸载程序

```java
 Intent intent = new Intent();
        intent.setAction(Intent.ACTION_DELETE);
        intent.setData(Uri.parse("package:" + packName));
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
```
