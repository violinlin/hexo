---
title: 云服务器配置Tomcat
photos:
   - '/img/pictures/picture9.jpg'
date: 2017-09-05 11:35:20
tags: [Server]
categories: [Server]
---


> 在[Apache Tomcat](http://tomcat.apache.org/)官网上选择适当的Tomcat版本，参考[云服务器配置JDK](http://blog.androidhuilin.wang/2017/09/03/%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%85%8D%E7%BD%AEJDK/)将Tomcat的安装包上传到服务器，并解压到相应目录(可以自定义)，这里我将Tomcat的包解压到了`/usr/local/tomcat/tomcat8/`目录下

<!--more-->

## 1. 解压好即可直接启动 

```
1. 启动命令
/usr/local/tomcat/tomcat8/bin/startup.sh
2. 关闭命令
/usr/local/tomcat/tomcat8/bin/shutdown.sh
```
Tomcat默认端口号是8080，如果服务器没没有开放该端口，需要使用下面方法开放端口

```
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```
出现`success`表示添加成功
然后更新防火墙

```
firewall-cmd --reload
```
重启防火墙

```
systemctl restart firewalld.service
```
测试Tomcat服务是否成功，浏览器输入

```
http://[云服务器外网ip]:8080
```
## 2. 配置启动脚本
> 每次通过命令的全路径来启动/关闭服务也很麻烦，那么可以通过一个脚本来实现命令启动`Tomcat`

创建脚本
执行下面代码进入脚本编辑

```
vi etc/init.d/tomcat
```
按`i`进入编辑模式，拷贝下面内容，按`esc`键退出编辑，输入`:wq`保存并退出`vim`

```
# !/bin/bash    
# Description: start or stop the tomcat    
# Usage:        tomcat [start|stop|restart]    
#    
export PATH=$PATH:$HOME/bin  
export BASH_ENV=$HOME/.bashrc  
export USERNAME="root"  
  
case "$1" in  
start)  
#startup the tomcat    
cd [tomcat安装目录]/bin  
./startup.sh  
;;  
stop)  
# stop tomcat    
cd [tomcat安装目录]/bin
./shutdown.sh  
echo "Tomcat Stoped"  
;;  
restart)  
$0 stop  
$0 start  
;;  
*)  
echo "tomcat: usage: tomcat [start|stop|restart]"  
exit 1  
esac  
exit 0 
```
为脚本添加执行权限

```
chmod +x /etc/init.d/tomcat
```
创建软连接

```
cd usr/bin
ln -s /etc/init.d/tomcat
```
配置完成后测试是否有效，输入下面命令看Tomcat启动日志

```
tomcat start
tomcat stop
tomcat restart
```
## 3.修改默认端口号
切换目录到`[tomcat根目录]/conf`,编辑`server.xml`修改`port`端口,然后重启tomcat。

```
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

