---
title: kotlin中init方法简介
photos:
  - /img/pictures/picture10.jpg
date: 2020-01-28 15:15:12
tags: [kotlin]
categories: [kotlin]
---


## init方法

> 刚开始学习Kotliln语法时一直没搞明白初始化代码块`init`方法的调用时机，当初看网上文章说`init`方法会比构造方法优先执行，类似`JAVA`中的构造代码块。但是这种说法并不准确，下面介绍下init方法的调用时机。

<!--more-->

```kotlin
 class Student {
        init {
            println("init invoke")
        }

        constructor() {
            println("constructor invoke")
        }
    }
```
> 代码结果如下,`init`确实要比构造器先执行

```
init invoke
constructor invoke
```


> 但还有一种情况：`init`代码块可以访问主构造方法中的参数(当初一直纠结如果`init`比构造器先执行，那它怎么能访问构造器中的参数呢？)

```kotlin
 class Student(name: String) {
        init {
            println(name)
        }
    }
```
> 后来查了一些资料，了解了`init`方法的调用时机。虽然`Kotlin`兼容java,但他们毕竟是两种语言，不能用`java`中构造代码块的方式理解`init`方法。

> Kotlin中分为主构造函数和次级构造函数两种，主构造造函数最多只有一个，次级构造函数可以有多个

```
class Student(var name: String) { //主构造函数
        // 次构造函数
        constructor(name: String, age: Int) : this(name)
    }
```
> `Kotlin`中主构造函数是类头部的一部分位于类名称之后，这种声明方式导致主构造函数没有初始化代码，`init`方法则为主构造函数的初始化代码。所以`init`代码中可以访问主构造函数的参数。另外`Kotlin`中要求次构造函数要委托给主构造函数方法（如果有主构造函数的话），所以`init`方法执行顺序上要早于次构造函数。

> 另外`Kotlin`中主构造函数也不是必须的，我的理解是虽然次构造函数不用显示委托给主构造函数，但是如果有主构造函数的代码块，即声明了`init`方法时，仍要先执行`init`方法

下面贴一下`Kotlin`代码，以及反编译成`java`后的代码

```Kotlin
 class Student(var name: String) { //主构造函数
        init {
            println("init 1")
        }

        init {
            println("init 2")
        }

        // 次构造函数
        constructor(name: String, age: Int) : this(name) {
            println("constructor")
        }
    }

```

反编译成`java`代码

```java
 public Student(@NotNull String name) {
         Intrinsics.checkParameterIsNotNull(name, "name");
         super();
         this.name = name;
         String var2 = "init 1";
         boolean var3 = false;
         System.out.println(var2);
         var2 = "init 2";
         var3 = false;
         System.out.println(var2);
      }

      public Student(@NotNull String name, int age) {
         Intrinsics.checkParameterIsNotNull(name, "name");
         this(name);
         String var3 = "constructor";
         boolean var4 = false;
         System.out.println(var3);
      }

```

> 反编译成`java`代码也可以发现，`init`方法并不会编译成构造代码块。`Kotlin`中的主构造函数和次构造函数在`java`中都会编译成普通的构造函数，编译后的次级构造函数通过`this()`方法调用编译后的主构造函数（`Kotlin`中次构造函数需要委托给主构造函数）。`init`方法中的代码也会被添加到编译后的主构造函数中，添加的顺序则和在`Kotlin`中声明的顺序相关。

