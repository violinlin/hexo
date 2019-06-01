---
title: Android Palette调色板使用简介
photos:
  - /img/pictures/picture1.jpg
date: 2018-05-31 17:42:39
tags: [MD,Android]
categories:
---

> Palette调色板可以提取出一张图片里面比较突出的颜色值，使用Palette提取颜色可以保证UI色彩上的统一性；另外遇到图文排列的布局，可以调整字体的颜色避免被背景颜色同化。这篇简单介绍下Palette的使用。

<!--more-->

# 引入Palette包

> Palette 是在v7的support包中使用前应该先导入依赖包

```gradle

compile 'com.android.support:palette-v7:27.1.1'

```

# 通过指定图片的Bitmap获取Palette对象

> 获取Palette首先需要有图片的Bitmap对象，另外对象获取有同步和异步两种方式，建议使用异步获取Palette对象

```kotlin
 Palette.from(bitmap).generate(object : Palette.PaletteAsyncListener{
                            override fun onGenerated(palette: Palette) {
                                  // 获取palette对象
                                
                                
                            }

                        })
```

# 通过`Palette`获取`Swatch`

> Palette的样本`Swatch`有柔和的`muted`、有活力的`vibrant`和`dominate`三种，其中`muted`和`vibrant`又包括正常、`dark`,`light`三种色调，所以如下总共有七种`swatch`样本

**PS:除了`dominate`样本,其余六种样本有可能会拿空，所以最好做判空处理**

```kotlin
 val mutedSwatch = palette.mutedSwatch //柔和的
 val darkMutedSwatch = palette.darkMutedSwatch // 暗色柔和的
 val lightMutedSwatch = palette.lightMutedSwatch // 亮色柔和的
 
 val vibrantSwatch = palette.vibrantSwatch // 有活力的
 val darkVibrantSwatch = palette.darkVibrantSwatch // 暗色有活力的
 val lightVibrantSwatch = palette.lightVibrantSwatch // 亮色有活力的
 
 val dominantSwatch = palette.dominantSwatch
```

# 通过`Swatch`获取颜色值

> 每种样本`swatch`可以获取下面的颜色值，通常可以通过`swatch.getRgb()`设置背景颜色，`swatch.getBodyTextColor()`设置字体颜色

```kotlin
swatch.getPopulation(): 样本中的像素数量
swatch.getRgb(): 颜色的RBG值
swatch.getHsl(): 颜色的HSL值
swatch.getBodyTextColor(): 主体文字的颜色值
swatch.getTitleTextColor(): 标题文字的颜色值

```

效果图如下

![](/img/pallete_show.png)