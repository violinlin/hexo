---
title: AndroidStudio使用中遇到的问题
date: 2016-08-17 18:37:44
tags: [Android]
photos:
- http://7xvvky.com1.z0.glb.clouddn.com/picture/picture4.jpg
---

关于AndroidStudio布局文件的预览问题

<!-- more -->

## 布局预览问题
今天打开AndroidStudio突然发现布局文件没有了预览，显示是这个样子的：
![](http://7xvvky.com1.z0.glb.clouddn.com/blog/as/problem/preview1.PNG)
后来发现发现是自己下载了API24的SDK Platform，AS在预览是会自动选择本地最新版的SDK版本。好了，那么解决问题的方法就是把预览的版本调低就可以了。
åç
![](http://7xvvky.com1.z0.glb.clouddn.com/blog/as/problem/list.png)

效果如下

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/as/problem/normal.png)