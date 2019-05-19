---
title: 云服务器配置JDK
photos:
  - '/img/pictures/picture18.jpg'
date: 2017-09-03 00:29:45
tags: [Server]
categories: [Server]
---


> 最近看到了郭神在公众号里的京东云服务器优惠的推送，所以先买了两个月的体验版折腾一下。下面记录下在云服务器下配置JDK环境和Tomcat服务的过程。

<!--more-->

## 1. 上传JDK的包到云服务器
> 在云服务器上配置JDK首先需要在[Oracle](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)上找到适合云服务器系统的JDK版本。连接云服务器，切换或新建目录到JDK的安装目录**安装目录可以自定义**。下面以两种形式将JDK的包上传到云服务器。

### 1.1 通过 `wget`命令
> wget命令用来从指定的URL下载文件。wget非常稳定，它在带宽很窄的情况下和不稳定网络中有很强的适应性，如果是由于网络的原因下载失败，wget会不断的尝试，直到整个文件下载完毕。如果是服务器打断下载过程，它会再次联到服务器上从停止的地方继续下载。这对从那些限定了链接时间的服务器上下载大文件非常有用。
来自: http://man.linuxde.net/wget
```
wget http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jdk-8u144-linux-x64.tar.gz
```
一般下载文件`wget命令`+下载URl就行了，但是我在下载JDK的时候不能正常下载。上网查询后得配置一些参数，命令如下。

```
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jdk-8u144-linux-x64.tar.gz
```
### 1.2 从本地上传安装包到云服务器
> 如果使用`wget`下载速度太慢，或者本地已经有了JDK的安装包，可以通过文件上传的方式将安装包上传到云服务器。这边我已mac为例上传问题。

打开`终端`选择`新建远程连接`
![](/img/upload-shell.png)

选择`安全文件传输`，输入服务器的IP

![](/img/connect.png)
输入主机名开始连接
![](/img/sftp_connect.png)

输入密码提示连接成功后开始上传文件

```
put 本地文件路径  远程主机路径
```
## 2.解压JDK安装包
> 把JDK的压缩包上传到服务器后，现在使用`tar`命令解压安装包
```
tar zxvf [解压文件]
```
## 3.配置JDK环境变量
> 在电脑上为了方便使用JDK的命令我们一般都会配置环境变量。服务器上也是需要配置的。vi命令是UNIX操作系统和类UNIX操作系统中最通用的全屏幕纯文本编辑器。修改`vi /etc/profile`文件，在最后添加JDK的配置路径。

```
#set jdk environment  
export JAVA_HOME=/usr/lib/jvm/jdk1.7.0_21  （此处根据你所安装jdk的路径实际来写）
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH   
```

## 4.查看JDK是否配置成功
> 到这里JDK的配置基本就完毕了，现在就是需要测试下配置又没有成功了

使环境变量配置生效
```
source /etc/profile  
```
检验JDK配置是否成功

```
java -version 
```
正常显示JDK版本信息则配置成功
```
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```
