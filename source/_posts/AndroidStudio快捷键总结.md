---
title: AndroidStudio快捷键总结
date: 2016-07-31 20:19:32
tags: [Android,AndroidStudio]
photos:
 - '/img/pictures/picture8.jpg'
---
> 开发过程中熟练使用快捷键可以提高我们的效率还有我们的逼格。AS中提供了很多的快捷方式，我也在探索学习中，这里和大家分享一下我在开发中经常使用的快捷键，大家有好用的也可以推荐。mac系统使用将`control`替换成`cmd`键** 此篇不定期更新 **

当初信誓旦旦说每周都写点东西，结果... ,哎！人总是想偷懒的。

<!--more-->

## 编辑部分

### 代码的整行复制`control+D`和删除`control+Y`

通常我们可以通过`shift+home`或者`shift+end`来选中整行代码，再进行操作。`control+D`和`control+Y`可以快速的复制或者删除整行代码，在开发中使用还是比较多的。mac是

![整行复制和删除](/img/keymap_control_d.gif)

### 全局变量`control+alt+F`和局部变量的提取`control+alt+V`

在提取变量的同时我们可以为变量命名

![全局变量的提取](/img/keymap_control_alt_f.gif)

![局部变量的提取](/img/keymap_control_alt_v.gif)

### 方法的提取 `control+alt+M`

当我们写的代码段需要复用的时候，可以将他们提取成一个方法。这时候 `control+alt+M`就节省了我们的时间。

![方法的提取](/img/keymap_control_alt_v.gif)

### 重命名`shift+F6`

命名难免会更改，`shift+F6`可以快速重命名，收了吧。

![重命名](/img/keymap_shift_F6.gif)

## 窗口部分

### 查看当前类的方法大纲`control+F12`、`cmd+F12`

弹窗显示当前类中的方法和变量，便于了解该类的结构

![方法大纲](/img/keymap_control_f12.gif)


### 查看当前类的继承树`control+H`

显示当前类的继承关系，了解类的结构

![类的继承树](/img/keymap_control_h.gif)

### 重载父类的方法` Mac:control + o/Win:ctrl + o`

![](/img/keymap_control_o.gif)

### 切换文件窗口`Mac : control + tab/ win: ctrl + tab`

![](/img/keymap_control_tab.gif)
## 查找部分

### 类名查找`cmd+N`和文件名查找`cmd+shift+N`

当工程较大时可以通过类名或者文件名快速查找指定文件。

![类或文件名查找](/img/keymap_cmd_n.gif)

### 代码的书签标记`F11`和书签的查看`shift+F11`

书签功能还是很好用的，和看书的时候加书签一样使用就行了。

![代码书签](/img/keymap_f11+shirft_f11.gif)

## 其他部分

### 查看方法的参数`control+p`

调用方法时通过`control+p`查看参数信息

![参看参数](/img/control_p.gif)