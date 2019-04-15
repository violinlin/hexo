---
title: 自定义View--绘制钟表
date: 2016-06-27 22:28:30
tags: [Android,自定义view]
photos:
 - '/img/pictures/picture2.jpg'


---
Android中为我们提供了很多的功能强大的视图控件，但是有些时候我们需要特定效果的控件。这时我们就需要自定义View来实现需要的效果。关于自定义视图的内容很多，我也在慢慢研究，这里绘制了一个简单的钟表视图和大家一起分享学习。

<!--more -->

## 关于自定义视图
一般来说自定义视图的方法主要有三种。每种方法都有他们特定的用处，自定义视图时我们需要根据视图的功能来考虑选择那种方法。

| 自定义View|     用途|   
| :-------- | --------:|
| 继承View重写onDraw()方法|   一般用来绘制一些不规则的自定义的效果控件| 
| 继承ViewGrop|   一般用来绘制自定义的布局，可以将多个控件封装在一起使用|
|继承Android提供的视图（如TextView）|一般用来扩展原有视图的一些自定义的功能|
## 绘制钟表控件
> 根据上面的分析可知绘制钟表这种不规则的视图控件需要采用继承View 重写onDraw()的方法来实现，在onDraw()方法中通过画笔绘制自定义的视图

#### 视图属性的定义和使用
在使用Android提供的一些控件时，会经常用到一些控件的特定属性例如TextView的text属性，ImageView的scaletype属性等。这些属性可以设置控件的显示效果。自定义View的同时需要自定义一些属性来更好的设置view的显示效果。

##### 属性的定义
在res-values-styles文件中自定义view需要的属性，为每种属性定义合适的名称和类型。

```xml
 <declare-styleable name="clock_view">
        <attr name="clock_border_color" format="color" />
        <attr name="clock_border_width" format="dimension" />
        <attr name="clock_border_padding" format="dimension" />
        <attr name="clock_drawRect" format="boolean" />
        <attr name="clock_back_color" format="color"/>
    </declare-styleable>
```
view属性的类型主要有下面几种

| 类型|    资源类型|   样例|
| :-------- | --------:| :------: |
| reference|   参考某一资源的ID|   app:background = "@drawable/icon" |
|color | 颜色类型 | app:textColor = "#00FF00" |
|boolean|布尔类型true/false|  app:focusable = "true"|
|demension|尺寸类型| app:layout_height = "42dp"|
|float|浮点类型|app:fromAlpha = "1.0" |
|integer|整型|app:framesCount = "12"|
|String|字符串类型| app:text="hello world"|
|fraction|百分数类型|app:pivotY = "300%" |
|enum|枚举类型| app:orientation = "vertical" |
|flag|位或运算| app:windowSoftInputMode = "stateUnspecified \| stateUnchanged　\|　stateHidden"|
**注意：**属性的使用可以指定多种类型

```xml
 <attr name="clock_border_color" format="color|integer" />
```
##### 属性的使用

使用自定义的属性需要添加自定义命名空间，名称可以自己取

```xml
 xmlns:app="http://schemas.android.com/apk/res-auto"
```

```xml
  <com.example.administrator.clockdemo.ClockView
        android:id="@+id/clock_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:clock_border_color="#b510c7"
        app:clock_border_padding="10dp"
        app:clock_back_color="#000000"
        app:clock_border_width="3dp" />
```
#### 重写onDraw()方法
View的工作流程主要包括measure、layout、draw三部分：
1. measure 测量视图的大小
2. layout 确定视图在父布局中的位置
3. draw 执行onDraw()方法，绘制视图内容

通过继承View实现自定义视图的方法，主要分析流程的第三部。

##### 获取和使用自定义的视图属性

前面我们定义并在布局文件中使用了自定义的属性，下面介绍怎么在代码中使用使这些属性。

```java
 //获取自定义视图的属性
        TypedArray array = context.obtainStyledAttributes(attars, R.styleable.clock_view);
        //获取自定义的属性值，并为其设置默认值
        mBorderColor = array.getColor
                (R.styleable.clock_view_clock_border_color, Color.parseColor("#ffffff"));
        senWidth = (int) array.getDimension(R.styleable.clock_view_clock_border_width, 6);
        minWidth = (int) (senWidth * 1.5);
        hourWidth = (int) (minWidth * 1.5);
        padding = (int) array.getDimension(R.styleable.clock_view_clock_border_padding, 10);
        isDrawRecr = array.getBoolean(R.styleable.clock_view_clock_drawRect, false);
        clockBackColor = array.getColor(R.styleable.clock_view_clock_back_color, Color.parseColor("#281f1f"));
        array.recycle();//回收TypedArray ，以便后面的资源使用
```

##### 适配横竖屏切换操作，在切屏时重新绘制
 
> 每个视图其实都是一个矩形Rect，我们可以确定一个矩形范围，在其内部绘制我们的视图

```java
 /**
     * 屏幕旋转是重新计算钟表尺寸
     *
     * @param w
     * @param h
     * @param oldw
     * @param oldh
     */
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mBounds = new RectF(0, 0, w, h);
        width = w;
        height = h;
        if (width < height) {
            radius = width / 2 - padding;
        } else {
            radius = height / 2 - padding;
        }
        smallLength = 10;
        largeLength = 20;
    }
```
##### 绘制表盘
表盘其实就是一个大的圆形，我们可以根据系统提供的方法直接绘制，圆心就是前面定义的矩形rect的中心

```java
  canvas.drawCircle(mBounds.centerX(), mBounds.centerY(), radius, mPaint);//绘制表盘
```
##### 绘制刻度
绘制表盘上的刻度线，这部关键是通过三角函数计算刻度线的起始位置坐标
```java
  //绘制表盘刻度
        for (int i = 0; i < 60; ++i) {
            start_x = radius * (float) Math.cos(Math.PI / 180 * i * 6);
            start_y = radius * (float) Math.sin(Math.PI / 180 * i * 6);
            if (i % 5 == 0) {
                end_x = start_x - largeLength * (float) Math.cos(Math.PI / 180 * i * 6);
                end_y = start_y - largeLength * (float) Math.sin(Math.PI / 180 * i * 6);
                drawTextView(canvas, i);
            } else {
                end_x = start_x - smallLength * (float) Math.cos(Math.PI / 180 * i * 6);
                end_y = start_y - smallLength * (float) Math.sin(Math.PI / 180 * i * 6);
            }
            start_x += mBounds.centerX();
            end_x += mBounds.centerX();
            start_y += mBounds.centerY();
            end_y += mBounds.centerY();
            mPaint.setStrokeWidth(senWidth);
            canvas.drawLine(start_x, start_y, end_x, end_y, mPaint);

        }
        mPaint.setStrokeWidth(senWidth);
        mPaint.setStyle(Paint.Style.FILL);
        canvas.drawCircle(mBounds.centerX(), mBounds.centerY(), 10, mPaint);//绘制表盘中心点
```
##### 绘制刻度文本
和刻度一样，利用三角函数计算文本的起点坐标
```java
 /**
     * 绘制表盘刻度文本
     *
     * @param canvas
     */
    private void drawTextView(Canvas canvas, int i) {
        String textTime;
        float textTranslate = textSize / 2;
        float text_x;
        float text_y;
        //钟表左右边刻度
        if (Math.sin(Math.PI / 180 * i * 6) > 0) {

            text_x = (float) (mBounds.centerX() + (radius * 0.9 - textTranslate) * (float) Math.sin(Math.PI / 180 * i * 6));
        } else if (Math.sin(Math.PI / 180 * i * 6) < 0) {
            text_x = (float) (mBounds.centerX() + (radius * 0.9 + textTranslate) * (float) Math.sin(Math.PI / 180 * i * 6));

        } else {
            text_x = (float) (mBounds.centerX() - textTranslate);

        }

        //钟表上下刻度
        if (Math.cos(Math.PI / 180 * 6 * i) > 0) {
            text_y = (float) (mBounds.centerY() + -(radius * 0.9 - textTranslate) * (float) Math.cos(Math.PI / 180 * i * 6));

        } else if (Math.cos(Math.PI / 180 * 6 * i) < 0) {
            text_y = (float) (mBounds.centerY() + -(radius * 0.9 + textTranslate) * (float) Math.cos(Math.PI / 180 * i * 6));

        } else {
            text_y = (float) (mBounds.centerY() - textTranslate);

        }
        mPaint.setTextSize(textSize);
        mPaint.setStrokeWidth(2);
        textTime = i == 0 ? "12" : i / 5 + "";
        canvas.drawText(textTime, text_x, text_y, mPaint);
    }

```
##### 绘制钟表走针
绘制时分秒指针的原理其实是相同的，这里以秒针为例，将当前表盘分成60份，获取当前时间的秒数，绘制当前时间时指针的位置（当为时针是则将表盘分为类12份）
```java
 /**
     * 绘制秒针
     *
     * @param canvas
     */
    private int second = 0;

    private void drawSecLine(Canvas canvas) {
        second = getCurrentSec();
        int angle = second * 6;
        float endX = (float) (mBounds.centerX() + (radius / 1.2) * (float) Math.sin(Math.PI / 180 * angle));
        float endY = (float) (mBounds.centerY() + -(radius / 1.2) * (float) Math.cos(Math.PI / 180 * angle));
        float startX= (mBounds.centerX() - 50 * (float) Math.sin(Math.PI / 180 * angle));
        float startY=  (mBounds.centerY() + 50 * (float) Math.cos(Math.PI / 180 * angle));
        mPaint.setStrokeWidth(senWidth);
        canvas.drawLine(startX,startY, endX, endY, mPaint);
    }
```
##### 刷新界面
每个1s重新绘制界面更新指针的位置

```java
  postInvalidateDelayed(1000);//每隔1s重新绘制，刷新界面
```

#### 效果图
![效果图](/img/ClockView.gif)
[Git源码下载](https://github.com/violinlin/ClockDemo.git)