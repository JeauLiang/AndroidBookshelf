# 【Android练级之路】Service {ignore=true}

[TOC]

# 大纲

![image](https://github.com/meetleong/AndroidBookshelf/blob/master/resources/Service.png)

# 生命周期

![image](https://github.com/meetleong/AndroidBookshelf/blob/master/resources/service_lifecycle.png)

+ onCreate()
+ onStartCommand()
+ onBind()
+ onUnbind()
+ onDestroy()



# Service和Activity通信

### Binder
### Broadcast


# onStartCommand()返回值

### START_STICKY
当系统因回收资源而销毁了 Service，当资源再次充足时自动启动 Service。而且再次调用 onStartCommand() 方法，但是不会传递最后一次的 Intent，相反系统在回调 onStartCommand() 的时候会传一个空 Intent，除非有未处理的 Intent 准备发送。

### START_NOT_STICKY
当系统因回收资源而销毁了 Service，当资源再次充足时不再自动启动 Service，除非有未处理的 Intent 准备发送。


### START_REDELIVER_INTENT
当系统因回收资源而销毁了 Service，当资源再次充足时自动启动 Service，并且再次调用 onStartCommand() 方法，并会把最后一次 Intent 再次传递给 onStartCommand()，相应的在队列里的 Intent 也会按次序一次传递。此模式适用于下载等服务。

### START_STICKY_COMPATIBILITY 
是START_STICKY的兼容版本，但是不能保证Service被清理后一定会重新调用onStartCommand方法。




# 前台服务
前台服务与普通服务最大的区别在于，它会一直有一个正在运行的图标在系统的状态栏

### 创建前台服务
```java

public class MyService extends Service {

    ...

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("MyService", "onCreate executed");
        Intent intent = new Intent(this, MainActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);
        Notification notification = new NotificationCompat.Builder(this)
                .setContentTitle("This is content title")
                .setContentText("This is content text")
                .setWhen(System.currentTimeMillis())
                .setSmallIcon(R.mipmap.ic_launcher)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                .setContentIntent(pi)
                .build();
        startForeground(1, notification);
    }

    ...

}

```

### 启动与停止前台服务
```java
Intent foregroundIntent = new Intent(this, ForegroundService.class);
startService(foregroundIntent); // 启动前台服务
stopService(foregroundIntent); // 停止前台服务
```

# IntentService
IntentService 是 Service 的子类，并且所有的请求操作都是在异步线程里。如果不需要 Service 来同时处理多个请求的话，IntentService 将会是最佳的选择。

使用该服务只需要继承并重写 IntentService 中的 onHandleIntent() 方法，就可以对接受到的 Intent 做后台的异步线程操作了。这个服务会在运行结束之后自动停止。

MyService：
```java
public class MyIntentService extends IntentService {

  public MyIntentService() {
    super("MyIntentService");
  }

  @Override
  protected void onHandleIntent(Intent intent) {
    //TODO: 耗时操作，运行在子线程中
    Log.d("MyIntentService","Thread is "+Thread.currentThread().getId());
  }

  @Override
  public void onDestroy() {
    super.onDestroy();
    Log.d("MyIntentService","onDestroy executed");
  }
}
```
MyActivity：
```java
public class MyActivity extends AppCompatActivity {

  protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.myactivity);

    Button btn = findViewById(R.id.btn);
    btn.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Log.d("MyActivity","Thread is "+Thread.currentThread().getId());
            Intent intent = new Intent(MyActivity.this,MyIntentService.class);
            startService(intent);
        }
    });
}

  @Override
  public void onDestroy() {
    super.onDestroy();
    Log.d("MyIntentService","onDestroy executed");
  }
}
```
点击按钮之后即可看到以下效果

![image](https://github.com/meetleong/AndroidBookshelf/blob/master/resources/ServiceComLogd.png)

# 服务保活

### 提高服务优先级

### 将服务改成前台服务
