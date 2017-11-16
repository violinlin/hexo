---
title: Service
photos:
  - 'http://oi2uynp9t.bkt.clouddn.com/IMG_31.JPG-blog'
date: 2016-09-18 23:13:58
tags: [Android基础]
categories: [Android]
---
>**认识Service** 无UI页面的，并长期处理后台运行，其所在的进程状态为服务进程 ,它本身在主线程执行。
>**它的主要作用** 用于管理生命周期较长的组件，如带有子线程管理的MediaPlayer、定时器等。

<!--more-->

# 创建新的Service

>* 创建Service的子类
>* 重写一个抽象方法（onBind( )），和三个核心方法的生命周期方法 onCreate()，onStartCommand()， onDestroy()  ，创建和销毁只执行一次，
>执行多少次content.startService()服务里的onStartCommand()就会被调用几次
>* service的两种启动方式
>1. context.startService：
>context.startService-->onCreat()(只执行一次)-->onstartCommand()-->Service Running-->context.stopService()-->onDestory()(只执行一次)-->Service Stop
>2. context.bindService(Intent serivce, ServiceConnection connection,int flags):
>context.bindService-->onCreat()(只执行一次)-->onBind()(只绑定一次)-->ServiceRunning-->onUnBind()-->onDestory()(只执行一次)-->Service Stop

>* 它可以通过Service.stopSelf()方法或者Service.stopSelfResult()方法来停止自己，只要调用一次stopService()方法便可以停止服务，无论调用了多少次的启动服务方法
>* 在清单文件中注册Service组件
>* 在Activity中，通过contxt对象启动和关闭服务Context.startService(intent),Context.stopService();

![service lifecycle](http://7xvvky.com1.z0.glb.clouddn.com/blog/service/service_lifecycle.png)

```
public class MyService extends Service {
	public void onCreate() { //只执行一次，用于初始化Service
		super.onCreate();
		Log.i("debug", "onCreate");
	}

	@Override
	public int onStartCommand(Intent intent, int flags, int startId) {
	//第一个参数intent用与传递参数
	// TODO 每次启动Service都会执行的方法，在此实现核心的功能
	//
		Log.i("debug", "onStartCommand");
		return super.onStartCommand(intent, flags, startId);
	}
	
	@Override
	public IBinder onBind(Intent intent) {
		return null;
	}
	
	@Override
	public void onDestroy() { //只执行一次，用于销毁Service组件
		super.onDestroy();
		Log.i("debug", "onDestroy");
	}

}

```
# BindeService
> 通过IBinder方式将两个组件进行绑定，然后互相传值，如果以绑定的方式启动服务，在解除绑定时也会自动停止服务

**bindService的过程**
 1. 在Service组件中，声明Binder类的子类（public），并在子类中声明组件之间的接口方法，（用于两个组件的交互）
 2. 在onBind()方法中，实例化Binder子类的对象，并返回
 3. 在绑定服务组件端（Activity），实例化ServiceConnection接口对象，并在此接口的onServiceConnected()方法中，将方法的第二个参数IBinder对象强转成Binder子类对象(回调方法)
 4. 调用 Context.bindService()来绑定服务。
 **通过bindService启动的Service不会调用onStartCommand()方法**

 >service：声明Binder的子类
```
public class MyService extends Service {
    @Override
    public void onCreate() {
        super.onCreate();
        Debug.debug("debug", "onCreate");
    }
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {    
        Debug.debug("debug", "onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }
    @Override
    public void onDestroy() {
        super.onDestroy();
        Debug.debug("debug", "onDestroy");
    }
    public IBinder onBind(Intent intent) {
        Debug.debug("debug", "onBind");
        2.实例化Binder的子类对象，并且返回
        return new MyBinder();
    }
    @Override
    public boolean onUnbind(Intent intent) {
        Debug.debug("debug", "onUnbind");
        return super.onUnbind(intent);
    }
1. 声明Binder类的子类，用于Service与绑定的Activity的组件进行通信
    public class MyBinder extends Binder {
        public void start() {
            Debug.debug("debug", "start");
        }

        public void stop() {
            Debug.debug("debug", "stop");
        }
    }


}

```
>Activity:实例化ServiceConnection对象回调Service中的方法

```
public class MainActivity extends ActionBarActivity {

    MyService.MyBinder myBinder=null;
   ServiceConnection connection=new ServiceConnection() {
       @Override
       public void onServiceConnected(ComponentName name, IBinder service) {
           myBinder= (MyService.MyBinder) service;
       }

       @Override
       public void onServiceDisconnected(ComponentName name) {
            myBinder=null;
       }
   };
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Debug.isDebug=true;
    }

    public void button(View view){
        switch (view.getId()){
            case R.id.bind:
                Intent intent=new Intent(this,MyService.class);
//通过bindService启动的Service不会调用onStartCommand()方法，在解除绑定时也会自动停止服务            
bindService(intent,connection,BIND_AUTO_CREATE);
                break;
            case R.id.unbind:
                unbindService(connection);
                break;
            case R.id.start:
                myBinder.start();
                break;
            case R.id.stop:
                myBinder.stop();
                break;

        }

    }
}
```
## 绑定的服务类型

| 名称      |     作用 |   
| :-------- | --------:| 
| BIND_AUTO_CREATE  |   绑定的service不存在时，会自动创建 | 
| BIND_ADJUST_WITH_ACTIVITY |  service的优先级别与根据所绑定的Activity的重要程度有关，Activity处于前台，service的级别高 | 
| BIND_NOT_FOREGROUND  |   Service永远不拥有运行前台的优先级 | 
| BIND_WAIVE_PRIORITY  |   Service的优先级不会改变 |  
| BIND_IMPORTANT&BIND_ABOVE_CLIENT  | 所绑定的Activity处于前台时，Service也处于前台BIND_ABOVE_CLIENT 指定内存很低的情况下，运行时在终止绑定的Service之前终止Activity|

# IntentService

> IntentService是带有子线程的服务组件，其内部使用了单线程池模式，

  当所有的任务执行完成后，会自动关闭本服务
 **其生命周期方法：**
 * onCreate()
 * onStartCommand()
 * 	onHandleIntent() 在子线程中执行的方法
 *  onDestroy()
 创建IntentService子类时，需要提供一个无参的构造方法且调用super(name)子线程的名称，如果没有无参数的构造方法清单文件注册Service时会报错

## 粘性Service

> 通过在onstartCommand()方法中返回不同的Service标记创建不同的粘性Service

| 标签     |     作用 |
| :-------- | --------:| 
| START_STICKY 粘性标识    |   如果service进程被kill掉，保留service的状态为开始状态，但不保留递送的intent对象。随后系统会尝试重新创建service，由于服务状态为开始状态，所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到service，那么参数Intent将为null | 
| START_NOT_STICKY 非粘性标识 | “非粘性的”。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务。| 
| START_REDELIVER_INTENT 带数据的粘性标识 | 重传Intent。使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入| 

