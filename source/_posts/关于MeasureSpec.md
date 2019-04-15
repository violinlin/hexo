---
title: 关于MeasureSpec
photos:
   - '/img/pictures/picture1.jpg'
date: 2016-12-04 22:49:05
tags: [Android,自定义view]
---
视图view的大小是通过measure测量获取得到的,关于视图的测量我们有必要了解MeasureSpec这个类,官方API是这样描述的：

<!--more-->

视图view的大小是通过measure测量获取得到的,关于视图的测量我们有必要了解MeasureSpec这个类,官方API是这样描述的：
> A MeasureSpec encapsulates the layout requirements passed from parent to child.Each MeasureSpec represents a requirement for either the width or the height. A MeasureSpec is comprised of a size and a mode. 

通过简单翻译我们可以了解到： MesaureSpec这个类封装了父视图对子视图的布局要求,每个MeasureSpec代表了一个宽度或者高度的要求。MeasureSpec由测量模式和测量大小组成。

其中测量模式有以下三种

| 测量模式      | 特性 |
| :-------- | --------:|
| UNSPECIFIED    |   The parent has not imposed any constraint on the child. It can be whatever size it wants.|
|EXACTLY |The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be. | 
| AT_MOST |The child can be as large as it wants up to the specified size.|

为了节省内存MeasureSpec的设计也是很巧妙的

> MeasureSpecs are implemented as ints to reduce object allocation.This class is provided to pack and unpack the &lt;size, mode&gt; tuple into the int.

MeasureSpec通过32位的int类型来表示，其中高两位用来表示测量模式，后面30位是测量大小

看一下MeasureSpec的代码片段：

```java
public static class MeasureSpec {
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
//三种测量模式，用int类型的高两位表示
        public static final int UNSPECIFIED = 0 << MODE_SHIFT;

        public static final int EXACTLY     = 1 << MODE_SHIFT;

        public static final int AT_MOST     = 2 << MODE_SHIFT;
  //通过传入测量尺寸和测量模式，自定义MeasureSpec      
  public static int makeMeasureSpec(int size, int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }
        //获取MeasureSpec的测量模式
  public static int getMode(int measureSpec) {
            return (measureSpec & MODE_MASK);
        }
        //获取MeasureSpec的测量大小
  public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }

}
```

