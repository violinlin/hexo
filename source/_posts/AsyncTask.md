---
title: AsyncTask
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_29.JPG-blog'
date: 2016-09-18 22:24:35
tags: [Android基础]
categories: [Android]
---
> AsyncTack是对Thread+Handler良好的封装，用来执行异步任务

<!--more-->

三种泛型参数类型分别代表“启动任务执行的输入参数”、“后台任务执行的进度”、“后台计算结果的类型”。在特定场合下，并不是所有类型都被使用，如果没有被使用，可以用java.lang.Void类型代替。

```
    public abstract class AsyncTask<Params, Progress, Result>  
```

# 一个异步任务的执行一般包括以下几个步骤：

1. execute(Params... params)，执行一个异步任务，需要我们在代码中调用此方法，触发异步任务的执行。

2. onPreExecute()，在execute(Params... params)被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。

3. doInBackground(Params... params)，在onPreExecute()完成后立即执行，用于执行较为费时的操作，此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。**在子线程中执行**

4. onProgressUpdate(Progress... values)，在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。

5. onPostExecute(Result result)，当后台操作结束时，此方法将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。


# 在使用的时候，有几点需要格外注意：

1.异步任务的实例一般在UI线程中创建。
（它的内部使用Handler实现，需要Looper对象）

2.execute(Params... params)方法必须在UI线程中调用。

3.不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

4.不能在doInBackground(Params... params)中更改UI组件的信息。

5.一个任务实例只能执行一次，如果执行第二次将会抛出异常。



## 补充
1. 使用execute(),启动异步任务，任务串行执行（一个执行完才会执行另一个任务）
2. 使用  task.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"Task"+i);启动异步任务，任务会并发执行（多个任务一起执行）