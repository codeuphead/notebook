# Service

## 基础

- startService() 回调 onStartCommand()，如果服务未创建，会先回调 onCreate()
- stopService() 回调 onDestroy()
- bindService() 回调 onBind() 方法，返回 IBinder 实例
- unbindService() 回调 onDestroy()
- stopSelf() 回调 onDestroy()

### 生命周期

如过 startService() -> bindService() -> bindService，只有在所有客户端都调用  unbindService() 之后，再调用 stopService() 才能销毁 service

## Bind 模式

### Service 实现

Step 1 创建 Binder 子类，定义需要暴露的接口，并实例化

```java
public class MyService extends Service {
    private MyBinder mBinder = new MyBinder();

    class MyBinder extends Binder{
        public void start(){

        }

        public void stop(){

        }
    }
}
```

Step 2 在 Service 实现的 onBind 方法中，返回该对象

```java
@Override
public IBinder onBind(Intent intent){
    return mBider;
}
```

### Activity 实现

```java
public class MainActivity extends Activity {
    private MyService.MyBinder mBinder;

    private ServiceConnection connection = new ServiceConnection(){
        @Override
        public void onServiceDisconnected(ComponentName name){
        }

        @Override
        public void onServiceConnected(ComponentName name, IBinder service){
            mBinder = (MyService.MyBinder) service;
            mBinder.start();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState){
        ...
        Intent intent = new Intent(this, MyService.class);
        bindService(intent, connection, BIND_AUTO_CREATE);
    }

    @Override
    protected void onStop(){
        ...
        unbindService(connection);
    }
}
```

- BIND\_AUTO\_CREATE 表示活动和服务绑定后，自动创建服务，这会使得 Service 的 onCreate 执行， 但是 onStartCommand 不会执行

## 前台服务

- 可以理解成“持有 notification 的服务”
- 优先级高，不易被回收
- 在 onCreate() 中创建通知，调用 startForeground()

```java
public class MyService extends Service {
    @Override
    public void onCreate(){
        Intent intent = new Intent(this, MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
        Notification notification = new NotificationCompat.Builder(this)
            .setContentTitle()
            .setContentText()
            .setWhen()
            .setCmallIcon()
            .build();
        startForeground(1, notification);
    }
}
```

## IntentService

- HandlerThread 实现，有序队列
- 任务完成后自我销毁
- 使用场景为：有序后台任务
- 实现 onHandleIntent()，在此方法中执行耗时任务
- 调用 startService() 启动