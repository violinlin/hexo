---
title: Charles--抓取https报文
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_50.JPG-blog'
date: 2018-04-17 16:27:42
tags: [Charles]
categories:
---

为了适配ios和Android两端的开发需求，后台将原来的接口升级为了https接口，这里记录下配置`Charles`抓取https接口协议报文的方法。使用下面方法请保证可以正常抓取http请求报文。

<!--more-->

## 在pc端安装`Charles`的根证书

> Help-->SSL Proxying --> Install Charles Root Certificate

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_help_proxy_setting.png)

> 在钥匙串中将证书添加信任

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_charles_proxy_ca.png)

## 在客户端安装`Charles`证书

> 请保证`Charles`可以正常抓取手机上的http报文，这里以小米5安装证书为例。
> Help-->SSL Proxying --> Install Charles Root Certificate......mobile Device or RemoteBrower

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_help_device.png)

> 获取证书的下载链接，在手机浏览器中输入链接下载证书

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_client_crt.png)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_android_down_crt.png)

> 在手机上安装证书
> 设置-->更多设置-->系统安全--> 从存储设备安装  选取刚刚下载的证书安装(名称可以随便取)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_android_crt_install.png)

### 证书安装失败问题

> 最开始我是通过小米5自带的浏览器下载证书，并且直接安装的，但是提示下面错误

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_android_crt_error.png)

> 最终问题解决是通过UC浏览器下载证书，并通过上面的方式安装。

## 设置代理

> 在Proxy--> SSL Proxying Settins中添加抓取连接的代理



![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_proxy.png)

> 以抓取简书的接口为例,在`Host`中配置域名，在`port`中配置端口。通常http的默认端口为80，https的默认端口为443。

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_ssl_proxying_settings.png)

> 抓取日志如下
![](http://7xvvky.com1.z0.glb.clouddn.com/blog/charles/https_charles_jiansu.png)




