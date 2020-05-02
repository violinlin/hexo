---
title: Service
photos:
   - '/img/pictures/picture22.jpg'
date: 2016-09-18 23:13:58
tags: [Android基础,Service]
categories: [Android]
---
>**认识Service** 
无UI页面的，并长期处理后台运行，其所在的进程状态为服务进程 ,它本身在主线程执行。
>**它的主要作用** 
用于管理生命周期较长的组件，如带有子线程管理的MediaPlayer、定时器等。

<!--more-->

## 通过`startService()`启动服务

> 新建Service的子类，重写一个抽象方法（onBind( )），和三个核心的生命周期方法 onCreate()，onStartCommand()， onDestroy()。通过日志观察服务的生命周期 。**Ps:服务需要到清单文件中注册** 


```java
public class MyService extends Service {

    private static final String TAG = MyService.class.getName();

    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG,"onCreate");

    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG,"onStartCommand");


        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG,"onDestroy");
    }
}

```
> 通过`startService()`启动服务，通过`stopService()`停止服务

```java
 startService.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(v.getContext(), MyService.class);
                startService(intent);

            }
        });
        findViewById(R.id.btn_stop_service).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent(v.getContext(),MyService.class);
                stopService(intent);
            }
        });

```
> 运行日志如下

```
4-12 14:56:26.707 28721-28721/com.violin.violindemo D/com.violin.service.MyService: onCreate
04-12 14:56:26.709 28721-28721/com.violin.violindemo D/com.violin.service.MyService: onStartCommand
04-12 14:58:36.542 28721-28721/com.violin.violindemo D/com.violin.service.MyService: onStartCommand
04-12 14:59:19.694 28721-28721/com.violin.violindemo D/com.violin.service.MyService: onDestroy
```

**服务的创建`onCreate()`和销毁`onDestory()`只执行一次，执行多少次`content.startService()`服务里的`onStartCommand()`就会被调用几次**

context.startService-->onCreat()(只执行一次)-->onstartCommand()-->Service Running-->context.stopService()-->onDestory()(只执行一次)-->Service Stop


## 通过`bindService()`启动服务
> 通过`startService()`方法启动服务后，服务将一直处于运行状态，但是具体的运行逻辑`Activity`控制不了。如果需要实现`Service`和`Activity`中间的交互，需要通过`bindService()`方式启动。

新建`MyService`的时候会重写`onBind()`方法，上面通过`startService()`方式启动并不会调用此方法，只用通过`bindService()`方法启动，在`Actiivty`和`Service`之间绑定时`onBind()`方法才会被调用。

新建`MyBindler`类继承`Binder`，在类中定义`Activity`操作`Service`的方法，例如现在要实现播放功能：

```java
 class MyBindler extends Binder {
        public void init() {

        }

        public void play() {

        }

        public int getProgress() {
            return 0;
        }

        public void stop() {

        }
    }
```
在`MyService`的`onBind()`方法中返回`MyBindler`对象

```java
...
 @Override
    public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind");
        MyBindler myBindler = new MyBindler();
        return myBindler;
    }
 ...   

```

在`Activity` 中定义`MyBindler`对象，并在服务绑定时完成初始化

```java
 private MyService.MyBindler mProgressBindler;

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            Log.d("whl", "onServiceConnected");
            mProgressBindler = (MyService.MyBindler) service;

        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            Log.d("whl", "onServiceDisconnected");

        }
    };
```

通过`bindService()`方法绑定服务
**通过bindService启动的Service不会调用onStartCommand()方法**

```java
 Intent intent = new Intent(v.getContext(), MyService.class);
 // 绑定服务的类型和作用在后面列表中列出
 bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
```
绑定服务成功后会回调`onServiceConnected()`方法，同时返回`Service`中的`MyBindler`对象。这时`Activity`就可以通过初始化好的`MyBinder`对象操控`Service`。

通过`unBindService()`方法解绑服务

```java
unbindService(mServiceConnection);
```
context.bindService-->onCreat()(只执行一次)-->onBind()(只绑定一次)-->ServiceRunning-->onUnBind()-->onDestroy()(只执行一次)-->Service Stop

**PS:通过`startService()`启动的服务需要通过`stopService()`方法关闭，也可以通过`stopSelf()`方法，功能和`startService()`相同；
通过`bindService()`启动的服务需要通过`unindService()`方法关闭；当一个服务即通过`startService()`方法启动又通过`bindService()`方法启动，那么关闭时也需要调用两种方式的关闭方法才能停止服务。**



### 绑定的服务类型

| 名称      |    作用 |   
| :-------- | --------:| 
| BIND_AUTO_CREATE  |   绑定的service不存在时，会自动创建 | 
| BIND_ADJUST_WITH_ACTIVITY |  service的优先级别与根据所绑定的Activity的重要程度有关，Activity处于前台，service的级别高 | 
| BIND_NOT_FOREGROUND  |   Service永远不拥有运行前台的优先级 | 
| BIND_WAIVE_PRIORITY  |   Service的优先级不会改变 |  
| BIND_IMPORTANT&BIND_ABOVE_CLIENT  | 所绑定的Activity处于前台时，Service也处于前台BIND_ABOVE_CLIENT 指定内存很低的情况下，运行时在终止绑定的Service之前终止Activity|


![service lifecycle](/img/service_lifecycle.png)


## IntentService

> IntentService是带有子线程的服务组件，内部通过HandlerThread实现

  当所有的任务执行完成后，会自动关闭本服务
 **其生命周期方法：**
 * onCreate()
 * onStartCommand()
 * 	onHandleIntent() 在子线程中执行的方法
 *  onDestroy()
 创建IntentService子类时，需要提供一个无参的构造方法且调用super(name)子线程的名称，如果没有无参数的构造方法清单文件注册Service时会报错
 
 **不建议通过bindService来启动IntentService,因为IntentService是在子线程中执行，而且bindService()启动不会执行onStartCommand()方法，没有了IntentService的设计初衷**

## 粘性Service

> 通过在onstartCommand()方法中返回不同的Service标记创建不同的粘性Service

| 标签     |     作用 |
| :-------- | --------:| 
| START_STICKY 粘性标识    |   如果service进程被kill掉，保留service的状态为开始状态，但不保留递送的intent对象。随后系统会尝试重新创建service，由于服务状态为开始状态，所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到service，那么参数Intent将为null | 
| START_NOT_STICKY 非粘性标识 | “非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务。| 
| START_REDELIVER_INTENT 带数据的粘性标识 | 重传Intent。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入| 

## 前台服务

> 后台的服务进程是很容易被回收的，为了保证服务的稳定运行，可以使用前台服务。前台服务会有一个正在运行的图标在系统的状态栏显示，类似于通知效果。当服务取消时，图标会自动消失。

和创建通知时的代码相似，不过不同于通知使用`NotificationManager`的`notify()`方法，使用前台服务需要使用`Service`的`startForeground()`方法。代码如下，可以在服务的`onCreate()`中设置前台服务

```kotlin
 String notifyChannel = "service";
        NotificationManager manager = (NotificationManager) getSystemService(
                Context.NOTIFICATION_SERVICE);
        /**
         * 8.0及以上系统需要配置NotificationChannel
         * 如果没配置，{@link #startForeground(int, Notification)}
         * 会抛出 invalid channel for service notification 异常
         *
         */
        if (Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            //channelname 用户可以在设置界面调整不同渠道的通知显示信息
            NotificationChannel notificationChannel = new NotificationChannel(notifyChannel,
                    "service", NotificationManager.IMPORTANCE_HIGH);

            manager.createNotificationChannel(notificationChannel);
        }

        Intent intent = new Intent(this,ServiceActivity.class);


        PendingIntent pendingIntent = PendingIntent.getActivity(this,0,intent,0);

        Notification preNTF = new NotificationCompat.Builder(getApplicationContext(),
                notifyChannel)//notifyChannel 需要和 NotificationChannel中设置相同，通知才会弹出
                .setContentTitle("service" )
                .setContentText("message")
                .setSmallIcon(getApplicationContext().getApplicationInfo().icon)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.drawable.test1))
                .setContentIntent(pendingIntent)
                .setStyle(new NotificationCompat.BigPictureStyle().bigPicture(BitmapFactory.decodeResource(getResources(), R.drawable.picture1)))
                .setAutoCancel(true)
                .build();


        startForeground(1000, preNTF);//每个notifyID 对应一条通知，如果id相同，会在原来通知面板上更新内容
```
**PS:使用前台服务需要在清单文件中申明`FOREGROUND_SERVICE`权限**

```xml
 <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```
