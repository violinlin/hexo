---
title: Android渠道打包
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_14.JPG'
date: 2017-04-27 17:46:32
tags: [Android]
---


> 今天运营同学要帮忙打渠道包做首发推广，渠道平台要求首发得按要求替换应用图标和闪屏界面。本来做好了挨个打包的准备，后来找到种渠道包替换资源的方法。AS真心强大啊。

<!--more-->

## 1. 添加渠道信息
1.1 在`app/build.gradle`文件中添加渠道信息

 如果清单文件中配置了统计渠道字段，替换相应平台的渠道号

```
  <meta-data
            android:name="CHANEL"
            android:value="${CHANEL}" />
```

```
 productFlavors {
        baidu {
            manifestPlaceholders=[CHANEL:"baidu"]
        }
        yingyongbao {
            manifestPlaceholders=[CHANEL:"yingyongbao"]
        }
    }
```

## 2. 替换渠道资源文件

在`app/src`目录下创建渠道包名，例如创建`baidu`,该目录和`main`目录同级。在目录下创建`main`中对应的资源文件，在打渠道包时资源会自动进行替换。下面对应用图标做了替换。
![enter image description here](http://7xvvky.com1.z0.glb.clouddn.com/blog/chanelAndroidChanel.png)

## 3. 打包

通过`build/Generate Signed APK` 打渠道包，可以全选，也可以打单个渠道。

![enter image description here](http://7xvvky.com1.z0.glb.clouddn.com/blog/chanelandroid_chanel_pack.png)


