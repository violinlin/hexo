---
title: Kotlin学习笔记
photos:
  - /img/pictures/picture1.jpg
date: 2018-06-04 18:06:29
tags: [Kotlin]
categories: [Kotlin]
---

> 

## Kotlin中方法的参数默认值


## 可变参数（方法的参数数量不定）

## 中缀函数

## 访问权限

## `==` 、 `===`和 `equals` 

## Lambda函数

> 所谓lambda函数本质上就是一段逻辑代码，不过这段逻辑代码可以当做变量传递给其它函数使用

lambda表达式的语法如下:表达式用`{}`包围，`->`左面为参数列表，右面为函数体

```kotlin
{ x: Int, y: Int -> x + y }
```
例如:

```kotlin
val sum = { x: Int, y: Int -> x + y }
println(sum(1,2))
3
```
