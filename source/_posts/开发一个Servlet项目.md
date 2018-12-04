---
title: 开发一个Servlet项目
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_51.JPG-blog'
date: 2018-05-18 15:01:54
tags: [servlet]
categories: [Server]
---

> 在开发项目过程中，客户端经常会遇到以下问题：1. 开发环境未能及时部署造成的开发阻塞问题；2. 一些界面依赖后台接口动态绘制UI,需要后台配合修改数据。
通常客户端会在本地写一些假数据用来展示界面，等开发环境部署后，在联调接口和优化接口请求逻辑同时得记得把假数据给移除掉。这样的话客户端会多做一些无用
功。后台正式开发完之前肯定都是些假数据，但是我们可以把假数据放到服务器上而不是客户端。所以，现在的需求就是：有一个服务可以按照接口协议接收客户端的上传报文
并按照上传报文返回相应的返回报文（这里的返回报文其实就是一段按着接口协议的静态文本）。当然，后台小伙伴可没空给我写这个，不过技术上还是给了很多帮助的。

<!-- more -->

## 开发环境配置

项目开发需要下面工具,这篇博客主要记录IDEA创建Servlet项目，所以工具的下载和环境变量的配置自己百度搜索。

- [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=mac)(需要下载Ultimate版)
- [Tomcat](https://tomcat.apache.org/)

## idea 创建Servlet项目

> 现在后台开发基本都使用IDEA了，而且跟AS是同一家公司的产品，界面风格快捷键操作也很像，对于Android开发者上手还是挺友好的。

### 新建Java Web项目

> 创建最简单的Servlet项目，不需要引用Spring、Struts等框架

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/servlet_create.png)

### 添加Tomcat依赖

> 项目创建成功后，需要引入Tomcat的依赖。在Project Structure中进行如下操作

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/import_tom_jar.png)

 ![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/import_tom_jar2.png)

### 完善项目配置

#### 新建Servlet类，处理报文逻辑

> 添加好Tomcat的依赖后，在工程的'src'目录下创建自己的包名，在包底下新建一个类继承`HttpServlet`。**保证上面引入Tomcat依赖成功，不然找不到`HttpServlet`类**

```java
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
       
    }
}

```
#### 修改web.xml 配置文件

> 在web-->WEB-INF-->web.xml 文件中添加下面信息

```xml
  <servlet>
        <servlet-name>FirstServlet</servlet-name>
        <servlet-class>com.first.FirstServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FirstServlet</servlet-name>
        <url-pattern>/firstservlet</url-pattern>
    </servlet-mapping>
```


### 配置Tomcat

> 如上，基本的项目框架已经完善。但是想让项目通过网络访问，还差最后一步，为项目配置Tomcat服务器。


![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/deploy_tomcat1.png)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/deploy_tomcat2.png)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/deploy_tomcat3.png)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/deploy_tomcat4.png)

日志显示下面表示服务启动成功

```
[2018-05-22 03:18:59,303] Artifact FirstServlet:war exploded: Artifact is deployed successfully
[2018-05-22 03:18:59,303] Artifact FirstServlet:war exploded: Deploy took 715 milliseconds
```

## 测试服务

> Tomcat服务启动成功后就可以通过`Postman`或者其他工具来测试服务接口

修改'FirstServlet'中的处理逻辑，处理上传报文和返回报文。这里只是做了简单处理：读取打印上传报文，添加自定义信息拼接上传报文最为回复报文返回。
实际可按照自己项目中的报文加解密规则处理。

```
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        InputStream inputStream = req.getInputStream();
        byte[] b = new byte[1024];
        int l = 0;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        while (((l = inputStream.read(b)) != -1)) {

            baos.write(b, 0, l);
        }
        String request = new String(baos.toByteArray());
        System.out.println("上传报文：" + request);

        resp.setContentType("text/html;charset=UTF-8");//解决返回报文中中文乱码问题
        resp.getWriter().println("回复报文开始----------------");
        resp.getWriter().println(request);
        resp.getWriter().println("回复报文结束----------------");
        resp.getWriter().close();

    }
}

```

访问路径为：本机ip地址+8080（tomcat端口）+/firstservlet(项web.xml中`url-pattern`字段)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/test_servlet1.png)

![](http://7xvvky.com1.z0.glb.clouddn.com/blog/firstservlet/test_servlet2.png)
