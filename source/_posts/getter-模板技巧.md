---
title: getter()模板技巧
photos:
  - '/img/pictures/picture10.jpg'
date: 2018-04-08 15:52:22
tags: [AndroidStudio]
categories:
---


> 通常在处理bean类的时候，我们会通过AS的模板功能添加setter()和getter()方法,这种操作相当方便。但有些时候不能适应我们所有的需求，下面记录下适应具体需求时的一些修改。

<!--more-->

## 定义全局变量时以'm'开头

> 有人喜欢在定义变量时加个前缀，用来区分全局变量、静态变量、局部变量等，例如常用的成员变量前缀为m。如果添加了前缀，在生成get\set方法时，方法名会带上这个前缀,如下

```java
 private String mName;

    public String getmName() {
        return mName;
    }
```

避免方法名携带前缀字段可以尝试用下面的方法

进入AS的`Preferences`界面，`Editor-->Code Style-->Java`,在'Name Prefix'中添加前缀字段，例如'm'

![](/img/getter.png)

重新生成'get'方法

```java
 private String mName;

    public String getName() {
        return mName;
    }
```

## Getter 模板修改--自动处理 null 判断

> 开发中会经常遇到实体类的一些属性未初始化造成的空指针异常，尤其是跟后台定义的接口实体类，由于解析或其他问题属性未初始化在调用时直接报空指针异常。
实体类中常见的有`String`类型和`List`类型，我们可以修改下`get`方法的模板，在使用属性时做一层非空处理。 [参考链接](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650825268&idx=1&sn=449a6d4a71872560fcb087f41e7ec7cc&chksm=80b7b6aab7c03fbc17a4cb6f1dd96ebfd99244c311fff2e3cb2cc613a3cfdfc1b0e12f71478a&scene=38#wechat_redirect)

修改后生成代码如下:

```java
 private String mName;
    private List<String> mList;

    public String getName() {
        return mName == null ? "" : mName;
    }

    public List<String> getList() {
        if (mList == null) {
            return new ArrayList<>();
        }
        return mList;
    }
```

修改生成`get`的模板方法如下

1. 进入生成`get`方法的弹窗界面，点击右上角进入模板编辑界面
 
 ![](/img/getter_select.png)
 
2. 新建代码模板,修改模板代码如下(可以复制原来模板中的内容主要修改了`$(name){…} `中的return规则)

 ![](/img/getter_templete.png)
 
 ```java
 #if($field.modifierStatic)
 static ##
 #end
 $field.type ##
 #set($name = $StringUtil.capitalizeWithJavaBeanConvention($StringUtil.sanitizeJavaIdentifier($helper.getPropertyName($field, $project))))
 #if ($field.boolean && $field.primitive)
   #if ($StringUtil.startsWithIgnoreCase($name, 'is'))
     #set($name = $StringUtil.decapitalize($name))
   #else
     is##
 #end
 #else
   get##
 #end
 ${name}() {
   #if ($field.string)
      return $field.name == null ? "" : $field.name;
   #else 
     #if ($field.list)
     if ($field.name == null) {
         return new ArrayList<>();
     }
     return $field.name;
     #else 
     return $field.name;
     #end
   #end
 }
 ```




