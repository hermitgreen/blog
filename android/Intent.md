# Intent

Intent是Android程序中，各组件交互的一种重要方式，主要用途为启动活动，启动服务，发送广播等

## Intent如何使用

### 显式的Intent

构造函数为 `Intent(Context packageContext, Class cls)`，第一个参数为一个Contxt，作为上下文，第二个参数为目标活动

### 隐式的Intent

构造函数只有一个字符串`Intent("com.example.activity.ACTION_START")`

## 使用Intent在活动之间穿梭

### 显式的Intent调用

```java
Intent intent = new Intent(FristActivity.this,SecondActivity.class);
startActivity(intent);
```

可以看到，显示地调用Intent，只需要构造一个Intent实例，传入上下文和目标活动，然后利用`startActivity`方法启动即可

### 隐式的Intent调用

隐式的Intent需要有一个Activity响应，在AndroidManifest.xml中配置`<intent-filter>`以响应

```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.activity.ACTION_START" />
        <category android:name="android.intent.category.DEFAULT " />
    </intent-filter>
</activity>
```

然后，只需要隐式的调用Intent的构造函数

```java
Intent intent = new Intent("com.example.activity.ACTION_START");
startActivity(intent);
```

注意，只有当action和category同时匹配，才能响应。那么为什么以上代码可以响应呢？ 注意到`android.intent.category.DEFAULT`，这是默认的category，调用`startActivity(intent)`时，会自动地将这个category添加到Intent中

每个Intent可以指定多个category

```java
Intent intent;
intent.addCategory("com.example.activity.CATEGORY");
```

注意要到`<intent-filter>`中添加

```xml
<category android:name="com.example.activity.CATEGORY"/>
```

## 更多隐式Intent的用法

### 举例：调用拨号界面

```java
Intent intent = new Intent(Intent.ACTION_DIAL);
intent.setData(Uri.parse("tel:10086"));
startActivity(intent);
```

打开网页也可以用这种方式实现

## 向下一个活动传递数据

首先利用`putExtra()`方法把想要传递的数据暂存在Intent中

```java
String data = "Message";
//显式调用
Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
intent.putExtra("Extra_Data",data);
startActivity(intent);
```

`putExtra()`接受两个参数，第一个是键，也就是用于取值的ID，第二个是值

然后在SecondActivity中把值取出来

```java
Intent previousIntent = getIntent();
String data = previousIntent.getStringExtra("Extra_Data");
//TODO
```

`getStringExtra()`的参数就是我们之前设置的键，除此之外，还有`getIntExtra()`等方法

## 返回数据给上一个活动

要返回数据给上一个活动，在启动时，必须使用`startActivityForResult()`方法，这个方法接受两个参数，第一个是Intent，第二个是请求码

请求码是一个唯一的值，用于在之后的回调中判断数据来源，这里使用了`1`作为请求码

```java
//FirstActivity
//显式调用
Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
startActivityForResult(intent,1);
//SecondActivity
Intent intent = new Intent();//这个Intent负责存储数据并带回上一活动
intent.putExtra("Data_Return","Message");
setResult(RESULT_OK,intent);
finish();//模拟该活动结束
```

`setResult()`方法用于向上一活动返回数据，接收两个参数，第一个参数表示处理结果，常用的值有`RESULT_OK`和`RESULT_CANCELED`两个，第二个就是带有数据的intent

在SecondActivity被销毁之后，使用`startActivityForResult()`启动的活动会回调FirstActivity中的`onActivityResult()`方法，因此需要重写这个方法并在其中处理数据

```java
@Override
protected void onActivityResult(int requestCode,int resultCode,Intent intent){
    switch (requestCode) {

        case 1:if(resultCode == RESULT_OK){
            String data = data.getStringExtra("Data_Return");
            //TODO
        }
        break;

        default:break;
    }
}
```

`onActivityResult()`方法有三个参数，第一个requestCode，就是我们在开启活动时传入的请求码，使用switch来判断是不是我们的相应活动。第二个resultCode，来判断是不是有数据传回，避免访问`null`。第三个就是带有数据的intent。

在实际应用中，用户常常是按BackPress返回上一个活动的，可以通过在SecondActivity中重写`onBackPressed()`方法来处理逻辑

```java
@Override
public void onBackPressed() {
    Intent intent = new Intent();//这个Intent负责存储数据并带回上一活动
    intent.putExtra("Data_Return","Message");
    setResult(RESULT_OK,intent);
    finish();
}
```

当用户按下返回按钮，就会执行`onBackPressed()`方法

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

### 本地广播

本地广播使用LocalBroadcastManager来对广播进行管理，并提供了自有的发送和注册广播的方法

```java
private LocalBroadcastManager localBroadcastManager;
Intent intent = new Intent("com.example.broadcast.LOCAL_BROADCAST");
LocalBroadcastManager.sendBroadcast(intent);
```