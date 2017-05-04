---
title: Banner的封装--实现ViewPager的循环轮播效果
date: 2016-07-18 00:08:34
tags: [Android,自定义view]
photos:
- http://7xvvky.com1.z0.glb.clouddn.com/picture/picture10.jpg
---
# 关于Banner
很多的项目中都需要实现Banner位滚动展示图片的效果，我这里使用了ViewPager和RadioGroup封装了一个带指示器功能的Banner视图，大家一起交流学习。

<!-- more -->

先上效果图

![banner.gif](http://upload-images.jianshu.io/upload_images/2352140-f5ef6bc5a797cb2a.gif?imageMogr2/auto-orient/strip)

# 自定义Banner视图

自定义视图的方法一般来说有三种，这里使用了继承ViewGroup(Relativelayout)的方法封装ViewPager和RadioGroup

## Banner布局文件的设置

``` XML
<merge xmlns:android="http://schemas.android.com/apk/res/android">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <android.support.v4.view.ViewPager
            android:id="@+id/viewpager"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

        <RadioGroup
            android:id="@+id/radiogroup"
            android:layout_width="match_parent"
            android:layout_height="10dp"
            android:layout_alignParentBottom="true"
            android:layout_centerHorizontal="true"
            android:layout_marginBottom="15dp"
            android:gravity="center"
            android:orientation="horizontal"></RadioGroup>
    </RelativeLayout>

</merge>
```
在构造方法中应用布局文件，初始化布局控件

``` java
   View view = LayoutInflater.from(context).inflate(R.layout.banner_layout, this, true);
        //滚动显示ViewPager
        viewPager = (ViewPager) view.findViewById(R.id.viewpager);
        radioGroup = (RadioGroup) view.findViewById(R.id.radiogroup);

```

## ViewPager的设置

### 适配器的设置

ViewPager本身支持视图的左右滑动，如果想实现无限轮播的效果需要在适配器中做一些设置。
> ViewPager从最后一位切换到首位会有一个快速滑动中间视图的效果，这样在显示效果上并不是太好。这里的处理方式是将适配器的数据源设置成一个最大值。

``` java
 /**
     * ViewPager的适配器
     */
    class BannerAdapter extends PagerAdapter {

        @Override
        public int getCount() {
            //将count设置成整数最大值，虚拟实现无线轮播
            return Integer.MAX_VALUE;
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            container.removeView(views.get(position % views.size()));
        }

        @Override
        public Object instantiateItem(ViewGroup container, int position) {
            View view = views.get(position % views.size());
            ViewGroup parent = (ViewGroup) view.getParent();
            //如果当前要显示的view有父布局先将父布局移除（view只能有一个父布局）
            if (parent != null) {
                parent.removeView(view);
            }
            container.addView(view);
            return view;
        }
    }
```
### 轮播效果的实现
通过Handler嵌套发送消息实现视图的切换效果
``` java
  /**
     * 通过嵌套发送消息循环滚动viewpager
     *
     * @param delayMillis 轮播图片的时间
     */
    public void bannerPlay(final long delayMillis) {
//        设置显示的初始位置，模拟向左无限滑动（设置合适的值就行，一般人也不会滑动很多）
        pagerPosition = views.size() * 100;
        if (handler == null) {
            handler = new Handler() {
                @Override
                public void handleMessage(Message msg) {

                    handler.post(new Runnable() {
                        @Override
                        public void run() {
                            viewPager.setCurrentItem(++pagerPosition);
                        }
                    });
                    Message message = handler.obtainMessage(0);
                    handler.sendMessageDelayed(message, delayMillis);
                }
            };
            Message message = handler.obtainMessage(0);
            handler.sendMessageDelayed(message, delayMillis);
        }
    }
```
## 数据源的设置和指示器的联动

### 数据源的设置
这里通过setData(Object ...data)来设置数据源，参数类型是Object的可变数组，可以传递图片的ID或者图片的链接等来设置数据源。
``` java
/**
     * 设置显示的资源
     *
     * @param data
     */


    public void setData(Object... data) {
        int id[] = new int[data.length];
        for (int i = 0; i < id.length; i++) {
            id[i] = i;
        }
        for (int i = 0; i < data.length; i++) {
            View view = LayoutInflater.from(getContext()).inflate(R.layout.banner_item_layout, viewPager, false);
            ImageView imageView = (ImageView) view.findViewById(R.id.iv_banner_item);
            if (data[i] instanceof Integer) {

                imageView.setImageResource((Integer) data[i]);
            } else {
//                TODO 如果设置的数据源为图片地址或者其他类型，根据具体情况为imageView设置图片源
            }
            views.add(view);
            //指示器小圆点的设置
            View indicator = LayoutInflater.from(getContext()).inflate(R.layout.indicator_item_layout, viewPager, false);
            radioButton = (RadioButton) indicator.findViewById(R.id.radioButton);
            radioButton.setId(id[i]);// 为radioButton设置一个id,不设置的话第一个radiobutton总是没法设置成选中中状态（不太清楚好像是个bug）
            if (i == 0) {
                radioButton.setChecked(true); //将第一个指示器设置为选中
            }
            RadioGroup.LayoutParams layoutParams = new RadioGroup.LayoutParams(dip2px(getContext(), 10f), dip2px(getContext(), 10f));
            layoutParams.leftMargin = dip2px(getContext(), 20);
            radioGroup.addView(radioButton, layoutParams);
        }
        bannerAdapter.notifyDataSetChanged();


    }


```
### 设置ViewPager和指示器的联动效果
``` java
    //设置viewpager的滚动监听
        viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
            }

            @Override
            public void onPageSelected(int position) {
                //改变当前显示的pager的位置
                pagerPosition = position;
                position = position % views.size();
                //改变被选中的指示器的状态
                for (int i = 0; i < radioGroup.getChildCount(); i++) {
                    ((RadioButton) radioGroup.getChildAt(i)).setChecked(i == position);
                }
            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });

```

[Git源码下载](https://github.com/violinlin/EndlessBanner.git)