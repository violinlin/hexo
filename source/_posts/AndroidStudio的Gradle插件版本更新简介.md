---
title: AndroidStudio的Gradle插件版本更新简介
photos:
  - 'http://7xvvky.com1.z0.glb.clouddn.com/picture/picture15.jpg'
date: 2016-09-21 13:53:50
tags: [Android,gradle]
---

> Android构建系统使用Android的Gradle插件通过Gradle的构建工具来支持构建Android程序。Android的Gradle插件独立于AndroidStudio运行，所以该插件和Gradle构建系统需要独立更新。

<!--more-->
## 更新Android的Gradle插件

**自动更新**
当你更新完Android Studio,你可能收到自动更新最新版插件的弹窗通知。你可以选择接受更新，或者自己根据项目需求指定插件版本
![](http://7xvvky.com1.z0.glb.clouddn.com/Blog/gradle/update.png)


**指定更新**
通过修改项目目录最顶部的`build.gradle`文件指定Gradle插件

```java
buildscript {
  ...
  dependencies {
    classpath 'com.android.tools.build:gradle:2.2.0'
  }
}
```
> **注意** 最好不要动态设置插件的版本号。例如：`com.android.tools.build:gradle:2.+` 使用这种方式可能会导致版本更新的混乱。

如果指定的插件版本还没有下载，Gradle会在你下次构建项目的时候下载，你也可以通过点击`Tools > Android > Sync Project with Gradle Files `来手动下载。

## 更新Gradle
**自动更新**
当你更新完Android Studio,你可能收到自动更新最新版Gralde的弹窗通知。你可以选择接受更新，或者自己根据项目需求指定Gradle版本
![](http://7xvvky.com1.z0.glb.clouddn.com/Blog/gradle/update.png)

**指定更新**
通过修改Gradle的分配引用文件`gradle/wrapper/gradle-wrapper.properties`指定Gradle版本，其实是修改了Gradle的下载链接，需要翻墙。也可以通过[这里](http://services.gradle.org/distributions)选择Gradle的版本下载，再复制到电脑的Gradle目录下。

1. Mac上会默认下载到/Users/<用户名>/.gradle/wrapper/dists
2. Windows默认下载到C:/Users//.gradlewrapper/dists

```
distributionUrl = https\://services.gradle.org/distributions/gradle-2.10-all.zip

```
## 通过Project Structure修改
也可以通过`File > Project Structure > Project`来修改修改Gradle版本`Gradle version`和插件版本`Andtoid Plugin Version `

![](http://7xvvky.com1.z0.glb.clouddn.com/Blog/gradle/Project%20Structure.png)


> **PS:** 使用中发现AndroidStudio应该有支持最高版本的Gradle限制，例如：在Android Studio 2.0上使用Gradle2.10是不起作用的，后来更新了AS问题解决了。

[原文地址](https://developer.android.com/studio/releases/gradle-plugin.html#updating-plugin)
