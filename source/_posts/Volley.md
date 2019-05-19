---
title: Volley
photos:
  - '/img/pictures/picture16.jpg'
date: 2016-09-17 21:38:58
tags: [Android基础]
categories: [Android]
---

> Volley特别适合数据量不大但是通信频繁的场景,对于大数据的加载并不适用

<!--more-->

## Volley请求json数据

1. 声明RequestQueue

```java
mRequestQueue =  Volley.newRequestQueue(this); 
```

2.  声明并使用Request

```java
    JsonObjectRequest jr = new JsonObjectRequest(Request.Method.GET,url,null,new Response.Listener<JSONObject>() {  
                @Override  
                public void onResponse(JSONObject response) {  
                    Log.i(TAG,response.toString());  
                    parseJSON(response);  
                    va.notifyDataSetChanged();  
                    pd.dismiss();  
                }  
            },new Response.ErrorListener() {  
                @Override  
                public void onErrorResponse(VolleyError error) {  
                    Log.i(TAG,error.getMessage());  
                }  
            });  
            //将请求添加到请求队列
            mRequestQueue.add(jr);  
```
> Volley提供了JsonObjectRequest、JsonArrayRequest、StringRequest等Request形式。


| 请求方式 | 返回对象 | 
| :-------- | --------:| 
| JsonObjectRequest |   返回JSON对象 |  
| JsonArrayRequest |   返回JsonArray |  
| StringRequest |   返回String |  

另外可以继承Request<T>自定义Request。

## Volley图片加载
### 使用ImageRequest下载图片
> Volley提供了多种Request方法，ImageRequest能够处理单张图片，返回bitmap。下面是ImageRequest的使用例子，和JsonRequest的一样。

```java
    singleImg=(ImageView)findViewById(R.id.volley_img_single_imgeview);  
            ImageRequest imgRequest=new ImageRequest(url, new Response.Listener<Bitmap>() {  
                @Override  
                public void onResponse(Bitmap arg0) {  
                    // TODO Auto-generated method stub  
                    singleImg.setImageBitmap(arg0);  
                }  
                //参数3/4: 内存中Bitmap 最大的宽/高度限制，用于降低内存的消耗
                 // 参数5：告诉 BitmapFactory，在生成Bitmap的时候，一个像素，包含的信息
                 //Config.ARGB_8888标识argb各占8个位，即一个像素4个字节
            }, 300, 200, Config.ARGB_8888, new ErrorListener(){  
                @Override  
                public void onErrorResponse(VolleyError arg0) {  
                    // TODO Auto-generated method stub  
                      
                }             
            });  
            mRequestQueue.add(imgRequest);  
```
### 使用ImageLoader加载图片
> ImageLoader这个类需要一个Request的实例以及一个ImageCache的实例。图片通过一个URL和一个ImageListener实例的get()方法就可以被加载。ImageLoader会检查ImageCache,如果缓存里没有图片就会从网络上获取。
> **ImageCache接口有两个方法，getBitmap(String url)和putBitmap(String url, Bitmap bitmap).这两个方法足够简单直白，他们可以添加任何的缓存实现。ImageCache和LruCache（最近最少使用算法）缓存一起使用缓存图片**
1. LruCache 构造⽅方法，如果 LruCache 重写了
sizeOf()，那么构造参数 代表 占⽤用的最⼤ 内存尺⼨；
2. 如果 LruCache 不重写 sizeOf（），代表 最多能够
存多少个对象；
```java
    RequestQueue mRequestQueue = Volley.newRequestQueue(this);  
            final LruCache<String, Bitmap> mImageCache = new LruCache<String, Bitmap>(20*1024*1024){
	 @Override
                protected int sizeOf(String key, Bitmap value) {

                    return value.getRowBytes()*value.getHeight();

                }

            };
};  
            ImageCache imageCache = new ImageCache() {  
                @Override  
                public void putBitmap(String key, Bitmap value) {  
                    mImageCache.put(key, value);  
                }  
      
                @Override  
                public Bitmap getBitmap(String key) {  
                    return mImageCache.get(key);  
                }  
            };  
            ImageLoader mImageLoader = new ImageLoader(mRequestQueue, imageCache);  
            // imageView是一个ImageView实例  
            // ImageLoader.getImageListener的第二个参数是默认的图片resource id  
            // 第三个参数是请求失败时候的资源id，可以指定为0  
            ImageListener listener = ImageLoader  
                    .getImageListener(imageView, android.R.drawable.ic_menu_rotate,  
                            android.R.drawable.ic_delete);  
            mImageLoader.get(url, listener);  
```

### 使用NetWorkImageView加载图片
> Volley中提供的ui控件，继承ImageView
> 这个控件在被从父控件detach的时候，会自动取消网络请求的，即完全不用我们担心相关网络请求的生命周期问题。
> 这里也会用到ImageLoadr

```java
setImageUrl(String url,ImageLoader imageLoader) 设置图片资源
setDefaultImageResId(int resId) 设置默认图片
```
## Volley
## Volley取消下载请求
> Activity被终止之后，如果继续使用其中的Context等，除了无辜的浪费CPU，电池，网络等资源，有可能还会导致程序crash，所以，我们需要处理这种一场情况。
使用Volley的话，我们可以在Activity停止的时候，同时取消所有或部分未完成的网络请求。
### 取消请求集合里的所有请求

```java
    @Override  
    public void onStop() {  
        for (Request <?> req : mInFlightRequests) {  
            req.cancel();  
        }  
        ...  
    }  
```

### 取消队列里的所有请求


```java
    @Override pubic void onStop() {  
        mRequestQueue.cancelAll(this);  
        ...  
    }  
``` 
### 根据RequestFilter或者Tag来终止某些请求
```java
    @Override public void onStop() {  
        mRequestQueue.cancelAll( new RequestFilter() {})  
        ...  
        // or  
        mRequestQueue.cancelAll(new Object());  
        ...  
```

## Volley的设计架构

> Volley使用了线程池来作为基础结构，主要分为主线程，cache线程和network线程。


主线程和cache线程都只有一个，而NetworkDispatcher线程可以有多个，如下图:
 ![Alt text](1444190962917.png)

