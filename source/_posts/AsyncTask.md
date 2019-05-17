---
title: AsyncTask
photos:
  - '/img/pictures/picture3.jpg'
date: 2016-09-18 22:24:35
tags: [Android基础]
categories: [Android]
---
> AsyncTack是对Thread+Handler良好的封装，用来执行异步任务

<!--more-->

三种泛型参数类型分别代表“启动任务执行的输入参数”、“后台任务执行的进度”、“后台计算结果的类型”。在特定场合下，并不是所有类型都被使用，如果没有被使用，可以用java.lang.Void类型代替。

```java
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
（它的内部使用Handler实现，需要Looper对象,也可以子线程中调用，后面会介绍到）

2.execute(Params... params)方法必须在UI线程中调用。

3.不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。

4.不能在doInBackground(Params... params)中更改UI组件的信息。

5.一个任务实例只能执行一次，如果执行第二次将会抛出异常。



## 补充
1. 使用execute(),启动异步任务，任务串行执行（一个执行完才会执行另一个任务）源码实现如下

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
> execute()方法的具体实现实在executeOnExecutor()方法中， onPreExecute()方法也在此处调用。任务的具体执行则在`sDefaultExecutor`中，这是一个`SerialExecutor`类型，源码如下

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
> 如上，`executeOnExecutor()`中调用` exec.execute(mFuture)`方法，具体执行在`SerialExecutor`的`public synchronized void execute(final Runnable r)`
方法中。它把每个任务添加到`mTasks`中，然后通过`scheduleNext（）`方法取出任务，并通过`THREAD_POOL_EXECUTOR`线程池来执行。每个任务的运行通过`try catch处理`，无论上个任务的运行结果，`scheduleNext()`都会在`finally()`中执行。
`SerialExecutor`用来保证任务是串行执行。



2. 使用  task.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR,"Task"+i);启动异步任务，任务会并发执行（多个任务一起执行）

> `executeOnExecutor()`启动方法会绕过`SerialExecutor`的逻辑，线程池会直接执行任务，并发的任务可以通过线程池控制，也可以自定义线程池

3. AsyncTask 不适合耗时任务

> AsyncTask 中任务的具体执行是通过 `threadPoolExecutor` 线程池，如下，非核心线程的超时时间为 3s,此外默认任务是串行执行，
如果多个耗时任务的话效率也会比较低。
```java
  private static final int CORE_POOL_SIZE = 1;
    private static final int MAXIMUM_POOL_SIZE = 20;
    private static final int BACKUP_POOL_SIZE = 5;
    private static final int KEEP_ALIVE_SECONDS = 3;
    
 static {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>(), sThreadFactory);
        threadPoolExecutor.setRejectedExecutionHandler(sRunOnSerialPolicy);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }

```
4. 子线程中是否可以启动AsyncTask？

> 可以启动，但是`onPreExecute`的方法执行在`execute()`的调用线程,`doInBackground()`方法仍然在线程池新建的线程中执行
> `onProgressUpdate()`、`onPostExecute()`等其他通过handler机制回调的方法则在主线程中执行。
> AsyncTask 是Handler和Thread的封装，创建任务时Looper是主线程的就不影响执行过程。不过最好还是按照官方文档在主线程中执行。
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
`onPreExecute`的方法执行在`execute()`的调用线程。

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

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }


```

