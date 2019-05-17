---
title: 关于EditText的使用技巧
photos:
  - '/img/pictures/picture10.jpg'
date: 2016-09-17 22:03:07
tags: [Android基础]
categories: [Android]
---

<!--more-->

# 键盘唤起后遮挡输入框

```
在清单文件的相应Activity下添加 android:windowSoftInputMode="adjustPan"属性

```
# 修改键盘右下角响应按键
> 修改成`完成`，也可以修改成`搜索`等

```
`android:imeOptions="actionDone"`
```

# 监听键盘右下角响应按键事件

> 以响应完成为例`actionId == EditorInfo.IME_ACTION_DONE`

```java
editText.setOnEditorActionListener(new TextView.OnEditorActionListener() {
                    @Override
                    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
                        if (actionId == EditorInfo.IME_ACTION_DONE) {
                            
                        return false;
                    }
                });
```