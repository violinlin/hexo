---
title: WebView使用总结
date: 2017-06-14 11:19:58
photos:
  - '/img/pictures/picture7.jpg'
tags: [Android]
categories: [Android]
---

> 1. WebView 实际上继承AbsoluteLayout,但是完全不支持ViewGroup的各种操作；无法查找里面的控件，也不能addView()
> 2. WebView内部的内容不是控件，而是屏幕绘制出来的内容，底层是一个浏览器引擎：webKit画出来的
> 3. webKit:一套开源的，支持多平台的浏览器引擎，几乎所有的android手机官方，默认浏览器，都使用了webkit ，能够将网页用图像的形式绘制出来，同时支持JavaScript和Html5的各种规范。

<!--more-->

## 1. WebView 加载方式

### 1.1 webView.loadUrl(String url)

```java
// 加载网络链接
webView.loadUrl("https://www.baidu.com/")
// 加载本地资源(test.html放在assets文件夹下)
webView.loadUrl("file:///android_asset/test.html")
```

### 1.2 webView.loadData(String data, String mimeType, String encoding)

用来加载html代码段，**可能会出现页面编码问题，设置编码格式`text/html;charset=UTF-8`**

```java
String summary = "<html><body>You scored <b>192</b> points.</body></html>";
 webview.loadData(summary, "text/html", null);
 // 标签中包含中文会出现编码问题，设置编码格式
  webView.loadData(summary, "text/html;charset=UTF-8", null);
```

### 1.3 webView.loadDataWithBaseURL(String baseUrl, String data,String mimeType, String encoding, String historyUrl)

和`loadData()`方法相似都是加载html代码段，但是多了`baseUrl,和historyUrl`，**因为html中`css`和图片等资源大都使用相对路径，定义了`baseUrl`可以成功加载这些资源。**

```java
   webView.loadDataWithBaseURL(testUrl, IOUtil.readFile(path), "html/text", null, testUrl);
```

   


## 2.WebSettings

> 下面的设置满足大多数情况下的WebView设置，注意的是'webSettings.setJavaScriptEnabled(true)'需要设置为true,除非界面是完全的静态界面不包含js代码，不然客户端加载网页会失败。

```java
  WebSettings webSettings = webView.getSettings();
         webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);//缓存模式，不加载缓存
         webSettings.setJavaScriptEnabled(true);// 网页支持JS脚本
         webView.getSettings().setDomStorageEnabled(true);// 开启DOM storage API 功能
         //允许混合内容 解决部分(5.0以上)手机 加载不出https请求里面的http的图片
         if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
             webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
         }
 
         //设置自适应屏幕，两者合用
         webSettings.setUseWideViewPort(true); //将图片调整到适合webview的大小
         webSettings.setLoadWithOverviewMode(true); // 缩放至屏幕的大小
 //
 //        //缩放操作
         webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
         webSettings.setBuiltInZoomControls(true); //设置内置的缩放控件。若为false，则该WebView不可缩放
         webSettings.setDisplayZoomControls(false); //隐藏原生的缩放控件
```



## 3. Java和JS的交互

> 设置WebView支持JS功能 `settings.setJavaScriptEnabled(true);`
在Html中定义需要调用的方法

```javascript
script type="text/javascript">
<!-- JS 调用 Java-->
    function showToastMessage() {
        window.android.showToastMessage(text)
    }
    <!--Java 调用 JS-->
    function alertMessage(message){
        alert(message)
    }
var text="js 调用 Java";
</script>
```
### 3.1 Java调用JS代码


#### 3.1.1. 通过WebView的loadUrl()方式

```java
 // 必须另开线程进行JS方法调用(否则无法调用)
                mWebView.post(new Runnable() {
                    @Override
                    public void run() {

                        // 注意调用的JS方法名要对应上
                        // 调用javascript的callJS()方法
                          webView.loadUrl("javascript:alertMessage(\"Java 调用 JS \")");
                    }
                });

```

#### 3.1.2. 通过WebView的evaluateJavascript()方式

> 1. 该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会
> 2. Android 4.4 后才可使用

```
mWebView.evaluateJavascript（"javascript:callJS(\"Java 调用 JS \")", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
```

### 3.2 JS调用Java代码

> 定义JS和Java的接口类, 需要在方法上面添加`@JavascriptInterface`注释

```java
//js和java的接口定义
    public class JsInteration {
        @JavascriptInterface
        public void showToastMessage(final String message) {
            jsHandler.post(new Runnable() {
                @Override
                public void run() {
                    Toast.makeText(MainActivity.this, message, Toast.LENGTH_SHORT).show();
                }
            });
        }
    }
```
> 添加JS交互接口,下面'android'为定义的接口名称，字段可以自定义

```java
 webView.addJavascriptInterface(new JsInteration(), "android");
```
> JS的调用方法,语法为`window.接口名.方法名(参数)`，接口名为上面定义的字段'android'。方法调用如下。

```javascript
 function showToastMessage() {
        window.android.showToastMessage(text)
    }
```
## 4.WebChromeClient的常用设置

> WebChromeClient是辅助WebView处理Javascript的对话框，网站图标，网站title，加载进度等.

监听网页加载进图，更新进度条
```java
 WebChromeClient chromeClient = new WebChromeClient() {
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                progressBar.setProgress(newProgress);

            }
        };

 webView.setWebChromeClient(chromeClient);
```

## 5.WebViewClient的常用设置

```java
 WebViewClient client = new WebViewClient() {
            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
                progressBar.setVisibility(View.VISIBLE);
                Log.d(TAG, "onPageStarted");
                if (mListener != null) {
                    mListener.onPageStart(url);
                }
            }

            @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                handler.proceed(); // 加载https自签名网站错误处理 例如https://www.12306.cn/mormhweb/
            }

            @Override
            public void onPageFinished(WebView view, String url) {

                super.onPageFinished(view, url);
                Log.d(TAG, "onPageFinished");
                progressBar.setVisibility(View.GONE);
                if (mListener != null) {
                    mListener.onPageFinish(url);
                }

            }

            @Override
            public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
                super.onReceivedError(view, request, error);
                Log.d(TAG, "onReceivedError");
                if (mListener != null) {
                    mListener.onPageError();
                }
            }

            @Override
            public boolean shouldOverrideUrlLoading(WebView webView, String s) {
            // 有些网页有拨打电话，打开第三方应用等的一些功能，如果有这种需求需要拦截请求地址，如果不是http
            、或者https协议，将请求交给系统处理。

                Log.d(TAG, "OverrideUrl: " + URLDecoder.decode(s));
                if (s.startsWith("http://") || s.startsWith("https://")) {
                    webView.loadUrl(s);
                } else {
                    try {
                        Uri uri = Uri.parse(s);
                        Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                        getContext().startActivity(intent);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }

                }
                return true;

            }
        };

```

## 6.清除界面

```java
 private void clearWebView() {
        if (webView != null) {
            webView.loadUrl("about:blank");
            webView.removeAllViews();
            webView.destroy();
        }
    }
```

## 7.其他设置

### 7.1拦截下载链接

```java
  webView.setDownloadListener(new DownloadListener() {
            @Override
            public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype, long contentLength) {
                // TODO 拦截下载请求

            }
        });
```
## WebView 的缓存问题

### 数据缓存

> 数据缓存分为AppCache和DOM Storage两种

#### AppCache

> 我们能够有选择的缓冲web浏览器中所有的东西，从页面、图片到脚本、css等等。 
尤其在涉及到应用于网站的多个页面上的CSS和JavaScript文件的时候非常有用。其大小目前通常是5M。 
在Android上需要手动开启（setAppCacheEnabled），并设置路径（setAppCachePath）和容量 
（setAppCacheMaxSize），而Android中使用ApplicationCache.db来保存AppCache数据！

```java
 webView.getSettings().setAppCacheMaxSize(1024*1024*8);
        String appCachePath = getContext().getApplicationContext().getCacheDir().getAbsolutePath();
        webView.getSettings().setAppCachePath(appCachePath);
        webView.getSettings().setAppCacheEnabled(true);
```
### DOM Storage

> 存储一些简单的用key/value对即可解决的数据，根据作用范围的不同，有Session 
Storage和Local Storage两种，分别用于会话级别的存储（页面关闭即消失）和本地化存储（除非主动 
删除，否则数据永远不会过期）在Android中可以手动开启DOM Storage（setDomStorageEnabled）， 
设置存储路径（setDatabasePath）Android中Webkit会为DOMStorage产生两个文件（my_path/localstorage/http_blog.csdn.net_0.localstorage和my_path/Databases.db)

```java
  // 开启DOM storage API 功能
        webView.getSettings().setDomStorageEnabled(true);
```

## 常见问题

### 加载带有js脚本的网页失败

webSettings.setJavaScriptEnabled(true);
设置WebView支持JS,如果不开启的话一些包含JS的网页可能会打开失败，例如[百度](https://www.baidu.com/)、[简书](https://www.jianshu.com/)。另外网页中的JS方法也会不响应。

### 下载链接点击无反应

> 需要自己实现DownLoadListener

```java
webView.setDownloadListener(new DownloadListener() {
            @Override
            public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype, long contentLength) {
                Uri uri = Uri.parse(url);
                Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                startActivity(intent);
            }
        });
```
### 加载https网站失败

> WebView加载自签名的https链接失败
需要在`WebViewClient`中重写下面方法

```java
          @Override
            public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
                handler.proceed(); // 加载https的不安全网站处理 例如https://www.12306.cn/mormhweb/
            }
```

### 使用了`Local Storage`存储的网页加载失败

> 如果网页使用了`Local Storage`存储，如果设置中没给与相应权限，界面会加载空白

```java
  // 开启DOM storage API 功能
        webView.getSettings().setDomStorageEnabled(true);
```



