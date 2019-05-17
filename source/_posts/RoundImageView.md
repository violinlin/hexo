---
title: RoundImageView
photos:
   - '/img/pictures/picture1.jpg'
date: 2018-03-14 11:21:06
tags: [Android,自定义view]
categories: [Android]
---
 
> 在项目中我们会经常用到圆形图片的情况，但是谷歌官方并没有提供这种视图控件，通常我们使用自定义视图来实现。
这个[CircleImageView](https://github.com/hdodenhof/CircleImageView)是我们以前项目中用到的一个
实现圆形图片的控件。最近有时间所以研究了一下源码，圆形图片主要是通过BitmapShader来实现，代码也比较容易理解对刚开始研究
自定义视图的同学还是很有帮助的。另外我也简单修改了源码实现圆角图片和圆形图片的功能。

<!--more-->

## 定义视图属性

>  明确自定义视图要实现的功能，并为其功能定义配置的属性信息。这里我想实现的是带有边框的圆角或者圆形的视图，并且边框的宽度、颜色、是否覆盖图片，圆角图片的圆角半径等功能都可动态配置。所以定义视图属性主要如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="RoundImageView">
        <attr name="round_border_width" format="dimension" /><!--边框宽度-->
        <attr name="round_border_color" format="color" /><!--边框颜色-->
        <attr name="round_border_overlay" format="boolean" /><!--边框是否覆盖图片-->
        <attr name="round_background_color" format="color" /><!--背景颜色-->
        <attr name="round_oval" format="boolean" /><!--圆形/圆角矩形-->
        <attr name="round_corner_radius" format="dimension" /><!--圆角半径-->
    </declare-styleable>
</resources>

```

## 自定义RoundImageView继承系统ImageView



### 画笔的初始化

> 整个视图我们可以理解为有三部分：背景、图片、边框。每一部分是独立绘制的，这里需要初始化三种类型的画笔。

```java
mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
        //图片画笔
        mBitmapPaint.setAntiAlias(true);
        mBitmapPaint.setShader(mBitmapShader);
        //边框画笔
        mBorderPaint.setStyle(Paint.Style.STROKE);
        mBorderPaint.setAntiAlias(true);
        mBorderPaint.setColor(mBorderColor);
        mBorderPaint.setStrokeWidth(mBorderWidth);
        //背景画笔
        mCircleBackgroundPaint.setStyle(Paint.Style.FILL);
        mCircleBackgroundPaint.setAntiAlias(true);
        mCircleBackgroundPaint.setColor(mCircleBackgroundColor);

```
上面图片画笔设置了`BitmapShader`为着色器。所谓着色器我是这样理解的，例如上面边框画笔设置了一个颜色值，画笔在视图画布上绘制的时候会让每个像素都绘制成相应颜色，直到绘制完整个视图。
如果像图片画笔那样设置了`BitmapShader`为着色器，画笔在绘制的时候会让着色器携带的图片绘制在视图上。如果图片大小不足以占满整个视图，画笔会根据`Shader.TileMode`来充满整个视图。
`CLAMP`用边缘色彩填充多余空间、`MIRROR`重复使用镜像模式的图像来填充多余空间 、`REPEAT`重复原图像来填充多余空间。

### 视图区域的计算

> 视图绘制区域的计算逻辑也比较好理解，主要做了兼容视图的padding属性的处理。另外，如果是圆形视图，绘制区域会以较短的边作为半径，既绘制区域内最大内切圆。 

```java
 /**
     * 计算可绘制的区域
     *
     * @return
     */
    private RectF calculateBounds() {
        int availableWidth = getWidth() - getPaddingLeft() - getPaddingRight();
        int availableHeight = getHeight() - getPaddingTop() - getPaddingBottom();
        if (mIsOval) {
            int sideLength = Math.min(availableWidth, availableHeight);
            /**
             * 如果是圆形图片，则选取较短的边作为直径，较长的一边位置居中
             */
            float left = getPaddingLeft() + (availableWidth - sideLength) / 2f;
            float top = getPaddingTop() + (availableHeight - sideLength) / 2f;

            return new RectF(left, top, left + sideLength, top + sideLength);

        } else {
            return new RectF(getPaddingLeft(), getPaddingTop(), getPaddingLeft() + availableWidth, getPaddingTop() + availableHeight);
        }

    }
```

上面计算的是视图内可绘制的最大区域，边框，图片、背景都应该在这个区域内绘制。不过属性中定义了`round_border_overlay`边框是否覆盖图片这个配置属性，所以当边框不是覆盖在图片上时，图片的可绘制区域应减少边框的宽度大小。

```java
  mBorderRect.set(calculateBounds());


        mDrawableRect.set(mBorderRect);
        if (!mBorderOverlay && mBorderWidth > 0) {
            /**
             * 当绘制圆角矩形时，圆角四周和border边框会有留白,所以适当增加图片的绘制区域
             */
            mDrawableRect.inset(mBorderWidth - 1f, mBorderWidth - 1f);
        }


```

### 图片的缩放处理

> 图片的可绘制区域确定后为了更好的显示效果，需要根据图片绘制区域和图片的大小进行缩放处理。


```java
 private void updateShaderMatrix() {
        float scale;
        float dx = 0;
        float dy = 0;

        mShaderMatrix.set(null);
/**
 *
 * 缩放值按照Math.max(Vw/Bw,Vh/Bh)，即图片的宽或高比上视图的宽或高最大值来计算，较小的一边通过计算偏移量居中。
 * 保证缩放后的图片占满视图。
 */
        if (mBitmapWidth * mDrawableRect.height() > mDrawableRect.width() * mBitmapHeight) {
            scale = mDrawableRect.height() / (float) mBitmapHeight;
            dx = (mDrawableRect.width() - mBitmapWidth * scale) * 0.5f;
        } else {
            scale = mDrawableRect.width() / (float) mBitmapWidth;
            dy = (mDrawableRect.height() - mBitmapHeight * scale) * 0.5f;
        }

        mShaderMatrix.setScale(scale, scale);
        mShaderMatrix.postTranslate((int) (dx + 0.5f) + mDrawableRect.left, (int) (dy + 0.5f) + mDrawableRect.top);

        mBitmapShader.setLocalMatrix(mShaderMatrix);
    }


```

`CircleImageView`的源码中是通过 `mBitmapWidth * mDrawableRect.height() > mDrawableRect.width() * mBitmapHeight`判断来计算缩放比的，刚开始一直没明白这种算法。后来又参考了网上其他图片缩放的一些算法明白了它的计算方式。
为了保证缩放后的图片能能够占满整个视图，缩放值scale=Math.max(视图高/图片高，视图宽/图片宽)，这个算法的变形就是上面的形式。

### 视图绘制

> 画笔、绘制区域、图片缩放等逻辑都处理好后，接下来就是视图的绘制。重写`onDraw()`方法。根据圆角或者圆形调用canvas.drawRoundRect或者canvas.drawCircle方法绘制视图。
绘制边框是需要注意，画笔设置了`setStrokeWidth`属性，绘制区域为绘制区域中心到`strokWidth`中心的距离，所以绘制边框时需要将绘制区域向内缩小`strokeWidth/2`的大小，保证边框完全显示。

```java
 if (mIsOval) {
            // 绘制圆形
            if (mCircleBackgroundColor != Color.TRANSPARENT) {
                canvas.drawCircle(mDrawableRect.centerX(), mDrawableRect.centerY(), mDrawableRect.width() / 2f, mCircleBackgroundPaint);
            }
            canvas.drawCircle(mDrawableRect.centerX(), mDrawableRect.centerY(), mDrawableRect.width() / 2f, mBitmapPaint);
            if (mBorderWidth > 0) {
//                绘制圆形的半径等于圆心到画笔宽度中心的距离
                canvas.drawCircle(mBorderRect.centerX(), mBorderRect.centerY(), mBorderRect.width() / 2f - mBorderWidth / 2f, mBorderPaint);
            }

        } else {
            // 绘制圆角矩形
            if (mCircleBackgroundColor != Color.TRANSPARENT) {
                canvas.drawRoundRect(mDrawableRect, mRadius, mRadius, mCircleBackgroundPaint);
            }
            canvas.drawRoundRect(mDrawableRect, mRadius, mRadius, mBitmapPaint);
            if (mBorderWidth > 0) {
                RectF rectF = new RectF(mBorderRect);
                //                绘制圆角矩形的宽/高等于矩形中心到上/下画笔宽度中心的距离
                rectF.inset(mBorderWidth / 2f, mBorderWidth / 2f);
                canvas.drawRoundRect(rectF, mRadius, mRadius, mBorderPaint);
            }
        }
        
```
## 效果预览

[源码地址](https://github.com/violinlin/ViolinDemo/blob/master/imageview/src/main/java/com/violin/imageview/RoundImageView.java)

![RoundImageView](/img/roundimageview.png)