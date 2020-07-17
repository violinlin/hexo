---
title: AsyncTask
photos:
  - '/img/pictures/picture13.jpg'
date: 2016-09-18 22:24:35
tags: [Android基础]
categories: [Android]
---

> AsyncTack是对Thread+Handler良好的封装，代码使用了`SerialExecutor`和`THREAD_POOL_EXECUTOR`两个线程池，分别用来做添加任务和执行任务的工作，使用了`InternalHandler`在线程之间通信。

<!--more-->

# 使用解析

AsyncTask的三种泛型参数类型分别代表“启动任务执行的输入参数”、“后台任务执行的进度”、“后台计算结果的类型”。在特定场合下，并不是所有类型都被使用，如果没有被使用，可以用java.lang.Void类型代替。

```java
    public abstract class AsyncTask<Params, Progress, Result>  
```

## 一个异步任务的执行一般包括以下几个步骤：

1. execute(Params... params)，执行一个异步任务，需要我们在代码中调用此方法，触发异步任务的执行。

2. onPreExecute()，在execute(Params... params)被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。

3. doInBackground(Params... params)，在onPreExecute()完成后立即执行，用于执行较为费时的操作，此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。**在子线程中执行**

4. onProgressUpdate(Progress... values)，在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。

5. onPostExecute(Result result)，当后台操作结束时，此方法将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。


## 在使用的时候，有几点需要格外注意：

1.异步任务的实例一般在UI线程中创建。
（它的内部使用Handler实现，需要Looper对象,也可以子线程中调用，后面会介绍到）

2.execute(Params... params)方法必须在UI线程中调用。

3.不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

4.不能在doInBackground(Params... params)中更改UI组件的信息。

5.一个任务实例只能执行一次，如果执行第二次将会抛出异常。



# 代码解析

## 使用execute(),启动异步任务，任务串行执行（一个执行完才会执行另一个任务）源码实现如下

```java
   public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
    
    
     @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }
    
    
```
> execute()方法的具体实现是在executeOnExecutor()方法中， onPreExecute()方法也在此处调用。具体任务会被封装在`mFuture`中，通过`SerialExecutor`来执行，看下`SerialExecutor`的源码。

```java
 private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
> `SerialExecutor`的`execute()`方法执行会首先将任务添加到`mTasks`阻塞队列里面，然后任务执行完成后，通过`scheduleNext()`方法，将下一个任务拿出，并通过`THREAD_POOL_EXECUTOR`线程池来做具体的执行逻辑。代码逻辑可以看到，一个任务执行完成后才会调用`scheduleNext()`方法继续下一个任务，**所以任务的执行是一个串行的逻辑**。

再看下`THREAD_POOL_EXECUTOR`的源码

```java
  private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));// 核心线程数[2、4]
  private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;// 线程池最大线程个数
  private static final int KEEP_ALIVE_SECONDS = 30;// 线程超时时间
  private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128); // 阻塞队列
            
  public static final Executor THREAD_POOL_EXECUTOR;

  static {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
  }
    
```


> `THREAD_POOL_EXECUTOR`的核心线程和最大线程数根据CPU个数来计算，核心线程数取值为[2,4]。阻塞队列大小为128个。

> 由于`SerialExecutor`会将任务串行提交，所以不适合做一些耗时的任务，不然很可能第一个任务还没执行完，程序退出了，导致后面的任务都没有执行。


## AsyncTask 任务能并行执行嘛？

> AsyncTask默认串行执行主要是因为`SerialExecutor`会将任务串行提交。看下`executeOnExecutor()`方法，可以在这指定线程池，绕开`SerialExecutor`串行逻辑。
 
```java
    task.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"Task"+i);//启动异步任务，任务会并发执行（多个任务一起执行）,也可以自定义线程池。  
```



## 子线程中是否可以启动AsyncTask？

> 可以启动，但是`onPreExecute`的方法执行在`execute()`的调用线程,`doInBackground()`方法仍然在线程池新建的线程中执行


```java
 @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

 @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();//onPreExecute的方法执行在execute()的调用线程。

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }


```

> `onProgressUpdate()`、`onPostExecute()`等其他通过handler机制回调的方法则在主线程中执行。

> AsyncTask 提供了三个构造方法，可供外部使用的只有无参的那个，具体初始化是在传参为Loop类型的构造器中。如代码，无论在哪个线程中启动，mHandler 中的Looper都是Looper.getMainLooper()。

```java
 /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     */
    public AsyncTask() {
        this((Looper) null);
    }

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     *
     * @hide
     */
    public AsyncTask(@Nullable Handler handler) {
        this(handler != null ? handler.getLooper() : null);
    }

    /**
     * Creates a new asynchronous task. This constructor must be invoked on the UI thread.
     *
     * @hide
     */
    public AsyncTask(@Nullable Looper callbackLooper) {
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);
            
            ...
            ...
            ...
}
```

```java
 private static Handler getMainHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
            // 通过handler传递值的方法都会在主线程中执行如`onProgressUpdate()`、`onPostExecute()`等
                sHandler = new InternalHandler(Looper.getMainLooper());
            }
            return sHandler;
        }
    }
```






