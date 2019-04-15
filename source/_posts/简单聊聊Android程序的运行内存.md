---
title: 简单聊聊Android程序的运行内存
photos:
  - '/img/pictures/picture1.jpg'
date: 2017-10-09 17:41:17
tags: [Android扩展]
categories:
---

> 每台手机允许程序的最大内存都不尽相同，如果你的设备已经root，可以查看系统文件获取；当然开发者也可以通过代码获取。

<!--more-->

## 在获取了root权限的设备上查看`/system/build.prop` 文件，下面是小米5的文件内容

```
dalvik.vm.heapsize=512m  
dalvik.vm.heapgrowthlimit=256m 
```
其中`heapgrowthlimit` 是普通应用的内存限制，当在清单文件中设置了`largeHeap=true`之后，可以使用的最大内存值

## 通过代码获取最大运行内存
> 可以试着修改清单文件中`largeHeap`的配置，看下每次的打印值。

```
 long maxMemory=Runtime.getRuntime().maxMemory()/1024/1024;
```

当然一般程序还是不建议设置`largeHeap=true`，我们更需要做的是内存的高效管理

