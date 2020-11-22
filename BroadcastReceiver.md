

# 【Android练级之路】Broadcast {ignore=true}

[TOC]


# 大纲

![image](https://github.com/meetleong/AndroidBookshelf/blob/master/resources/Broadcast.png)


Broadcast（广播） 是 Android 的四大组件之一，用于进程/线程间通信。

广播最大的特点就是发送方并不关心接收方是否接到数据，也不关心接收方是如何处理数据的，它只负责「说」而不管你「听不听」。

广播可以来之系统，例如，Android 系统在发生各种系统事件时发送广播（系统启动或者设备开始充电时）。

也可以来自于其他应用程序，例如，应用程序也可以发送自定义广播，来通知其他应用程序接受他们可能感兴趣的内容（更新数据）。



## 按发送方式分类

### 标准广播

+ 完全异步的广播，效率高，无法被截断
+ 发送 `sendBroadcast(intent)`

### 有序广播
+ 同步执行的广播，有先后次序，可以被截断
+ 发送 `sendOrderBroadcast(intent,null)`
+ 截断 `abortBroadcast()`


## 按注册方式分类
### 动态广播
+ 注册 `registerReceiver(BroadcastReceiver,intentFilter)`
+ 注销 `unregisterReceiver(BroadcastReceiver)`
### 静态广播
在manifest.xml中添加receiver和intent-filter

```xml
<receiver
    android:name=".MyReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <!-- 例如：接收系统开机广播 -->
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <!-- 例如：接收自定义的广播 -->
        <action android:name="com.demo.broadcast.MyReceiverFilter" />
    </intent-filter>
</receiver>
```


## 按定义方式分类
### 系统广播
上述的几种都属于系统广播，即发出的广播可以被其他任何应用程序接收到

### 本地广播
+ 只能在该应用程序内部传递
+ 获取实例 `LocalBroadcastManager LBM = LocalBroadcastManager.getInstance(this)`
+ 发送广播 `LBM.sendBroadcast(intent)`
+ 注册监听 `LBM.registerReceiver(Receiver,intentFilter)`
+ 注销监听 `LBM.unregisterReceiver(Receiver)`
