layout: title
title: Android 资源分包
photos:
   - '/img/pictures/picture6.jpg'
date: 2017-04-06 15:36:57
tags: [Android]
---

> 一直以为Android项目中的资源文件只能放在`src/main/res`的各个固定目录中。项目中的`java`代码可以根据功能模块创建不同的包来划分层次，资源文件则一般都放在`res`目录下。虽然可以通过命名规则区别不同模块的资源文件，但是随着业务的增长，开发人员的变动`res`目录会变得越来越繁杂。今天看到了一种资源分包的做法，感觉还不错，所以记录分享一下。

<!--more-->

在`build.gradle`文件中可以配置资源文件的文件路径

```gradle
sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res', 'res-ptr']
            assets.srcDirs = ['assets']
            jniLibs.srcDirs = ['libs']
        }
    }
```
如上清单文件、jar等都可以自己指定目录，如下配置资源文件目录

```gradle
sourceSets {
        main {
            res.srcDirs = ['src/main/res','src/main/res/mylayouts']
        }
    }
```
* **src/main/res**为项目默认路径
* **src/main/res/mylayout**为自定义路径（文件夹标识系统自动识别）
>  这种方式可以指定资源等文件夹，不过子文件夹得跟原来保持一致。例如图片文件放在`mylayouts/drawable`下、布局文件放在`mylayout/layout`下。

![](/img/packandroidrespack.png)

