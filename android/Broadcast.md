# Broadcast

广播是Android的一个重点，分为标准广播和有序广播

- 标准广播

标准广播(Normal broadcasts)完全异步执行，所有接收器同时接收到广播，效率较高。同时无法被截断

- 有序广播

有序广播(Ordered broadcasts)同步执行。优先级高的接收器优先接到广播，并有权限截断广播

## 动态注册广播接收器

首先我们看一下怎么用IntentFilter动态注册广播监听网络变化

首先声明全局变量

```java
private IntentFilter intentFilter;
private NetworkChangeReceiver netWorkChangeReceiver;
```

NetworkChangeReceiver是继承BroadcastReceiver的内部类，用于处理逻辑，即我们的广播接收器

```java
class NetWordChangeReceiver extends BroadcastReceiver{
    @Override
    public void onReceiver(Context context,Intent intent){
        ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo networkInfo = connectionManager.getActivityNetworkInfo();
        //TODO
    }
}
```

`getSystemService()`方法用于获得ConnectivityManager，其是一个系统服务类，主管网络连接，通过它的`getActivityNetworkInfo();`方法，得到NetworkInfo，NetworkInfo的常用方法有`isAvailable()`，返回一个布尔值

在`onCreate()`函数中注册广播

```java
intentFilter = new IntentFilter();
intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
netWorkChangeReceiver = new NetworkChangeReceiver();
registerReceiver(netWorkChangeReceiver,intentFilter);
```

`registerReceiver()`方法的第一个参数是广播接收器，第二个参数就是带有Action的intentFilter，这样，广播接收器就可以接受到相应Action的广播

需要注意的是，为了节省用户电池的电量，动态注册的广播接收器在结束活动时必须取消注册

```java
unregisterReceiver(netWorkReceiver);
```

在高版本的安卓里访问系统的状态需要声明权限，在AndroidManifest.xml中加入

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

## 静态注册广播接收器

静态注册主要用于在程序未启动的情况下接受广播

以接收开机广播为例，首先在AndroidManifest.xml加入

```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED">

<receiver
    android:name=".BootCompletedReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED">
    </intent-filter>
</receiver>
```

在相应的BootCompletedReceiver类中，只需要在`onReceive()`方法中处理逻辑

## 使用Intent发送广播

### 发送标准广播

以隐式的调用Intent的构造函数为例

```java
Intent intent = new Intent("com.example.broadcast.MY_BROADCAST");
sendBroadcast(intent);
```

Intent的参数，就是我们要发送的广播

注意：在安卓8.0及以上的系统中，必须显式地调用Intent的构造函数

```java
Intent intent = new Intent(MainActivity.this,MyBroadcastReceiver.class);
intent.setAction("com.example.broadcasttest.MY_BROADCAST");
sendBroadcast(intent);
```

Intent的第一个参数是上下文，第二个参数是接收器的类

显然不能调用其他程序接收器的类，于是，在安卓8.0 以上，建议使用动态注册发送跨程序广播

### 发送有序广播

有序广播只需要使用`sendOrderdBroadcast()`方法即可

```java
sendOrderdBroadcast(intent,null);
```

第二个参数是权限设置

在AndroidManifest.xml中可以配置有序广播的先后顺序

```xml
<intent-filter android:priority="100">
    <action android:name="com.example.broadcasttest.MY_BROADCAST" />
</intent-filter>
```

如果在MyBroadcastReceiver的`onReceive()`方法中调用了`abortBroadcast()`方法,则截断这条广播

## 本地广播

本地广播使用LocalBroadcastManager来对广播进行管理，并提供了自有的发送和注册广播的方法

声明变量

```java
private IntentFilter intentFilter;

private LocalReceiver localReceiver;

private LocalBroadcastManager localBroadcastManager;
```

localReceiver是继承BroadcastReceiver的内部类，用于处理逻辑，即我们的广播接收器

```java
class LocalReceiver extends BroadcastReceiver{
    @Override
    public void onReceiver(Context context,Intent intent){
        //TODO
    }
}
```

而我们的主角LocalBroadcastManager，由于这是一个服务类，故通过`getInstance()`方法获取实例

```java
localBroadcastManager = LocalBroadcastManager.getInstance(this);
```

发送广播的方法唯一不同的是调用了LocalBroadcastManager的`sendBroadcast()`方法

```java
Intent intent = new Intent("com.example.broadcast.MY_BROADCAST");
LocalBroadcastManager.sendBroadcast(intent);
```

同理，注册接收器也同样地使用LocalBroadcastManager的`registerReceiver()`方法

```java
intentFilter = new IntentFilter();
intentFilter.addAction("com.example.broadcast.MY_BROADCAST");
localReceiver = new LocalReceiver();
LocalBroadcastManager.registerReceiver(localReceiver,intentFilter);
```

不要忘了在结束程序的时候取消注册

```java
LocalBroadcastManager.unregisterReceiver(netWorkReceiver);
```