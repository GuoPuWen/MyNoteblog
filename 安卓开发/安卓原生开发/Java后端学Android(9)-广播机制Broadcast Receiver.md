本系列为《Android 第一行代码第二版》读书笔记

# 广播机制

安卓中的广播机制比较灵活，能够实现Android中的每一个应用程序都可以对自己感兴趣的广播进行注册，这样程序就可以只接受到自己所关心的广播内容，这些广播可以只来自系统的，也可以来自其他应用程序，Android提供了一套完整的API，运行应用程序自由的发送和接收广播

广播有两种类型：有序广播和标准广播：

- 标准广播：标准广播是一种完全异步执行的广播，在广播发出去之后，所有的广播接收器几乎都会同一时刻接收到这条广播消息，因此它们之间没有任何的先后顺序可言，这种广播的效率比较高，但是是无法被截断的

![image-20210509153706307](http://cdn.noteblogs.cn/image-20210509153706307.png)

- 有序广播：是一种同步执行的广播，在广播发出去之后，同一时刻只会有一个广播接收器能够收到这条消息，当这个广播接收器中的逻辑执行完毕之后，广播才会继续传递，所以这时候的广播接收器是有先后顺序的，并且前面的广播接收器还可以阶段正在传递的广播，这样后面的广播就无法收到广播消息

![image-20210509153932882](http://cdn.noteblogs.cn/image-20210509153932882.png)

### 动态注册

广播接收器可以自由的对自己感兴趣的广播进行注册，注册广播一般有两种方式：动态注册（在代码中注册）、静态注册（在AndroidManifest.xml中注册）

下面是一个动态注册的监听网络变化的程序

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private  IntentFilter intentFilter;

    private NetWorkChangeReceiver netWorkChangeReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        intentFilter = new IntentFilter();
        netWorkChangeReceiver = new NetWorkChangeReceiver();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        registerReceiver(netWorkChangeReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(netWorkChangeReceiver);
    }

    class NetWorkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            ConnectivityManager connectivityManager = (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo activeNetworkInfo = connectivityManager.getActiveNetworkInfo();
            if(activeNetworkInfo != null && activeNetworkInfo.isAvailable()){
                Toast.makeText(context, "Network is available", Toast.LENGTH_LONG).show();
            }else{
                Toast.makeText(context, "Network is unavailable", Toast.LENGTH_LONG).show();
            }

        }
    }
}
```

需要注意的是，Android为了保护用户设备的隐私和安全，规定了程序需要进行一些对用户来说比较敏感的操作，必须在配置文件中声明权限才可以，上面监听了网络的变化，所以必须在AndroidManifest.xml配置权限才能访问系统网络状态

AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.broadcasttest">
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BroadcastTest">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### 静态注册

动态注册的广播可以自由控制注册于主注销，很大的灵活性，但是缺点是必须在程序启动之后才能接受到广播，因为注册的逻辑是写在onCreate()里面的，使用静态注册可以让程序在未启动的情况下接受到广播

静态注册需要在AndroidManifest.xml进行注册，使用receiver标签，并告诉这个receiver注册哪一个action，下面是一个开机启动接受广播的案例

BootCompleteReceiver.java

```java
public class BootCompleteReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        Toast.makeText(context, "Boot Complete", Toast.LENGTH_LONG).show();
    }
}
```

AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.broadcasttest">

    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BroadcastTest">
        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>

        </receiver>

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

# 发送自定义广播

### 发送标准广播

发送广播使用intent进行发送，首先需要准备一个接收器用于接受发送的广播

MyBroadcastReceiver.java

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e("MyBroadcastReceiver", "广播接收");
        Toast.makeText(context, "Received in MyBroadcastReceiver", Toast.LENGTH_LONG).show();
    }
}
```

AndroidManifest.xml

```xml
<receiver android:name=".MyBroadcastReceiver"
          android:enabled="true"
          android:exported="true">
    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>

</receiver>
```



MainActivity.java

```java
public class MainActivity extends AppCompatActivity {



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = findViewById(R.id.button1);
        button.setOnClickListener((View v) -> {
            Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST");
            //参数1:包名；参数2:接收器的路径
            intent.setComponent(new ComponentName("com.example.broadcasttest","com.example.broadcasttest.MyBroadcastReceiver"));
            Log.e("MainActivity","按钮点击");
            sendBroadcast(intent);
        });
    }
    
}
```

上面案例用于每次点击一下按钮就会发送一个"com.example.broadcasttest.MyBroadcastReceiver"类型的广播，而MyBroadcastReceiver就会进行接受，看日志打印

![image-20210509225838772](http://cdn.noteblogs.cn/image-20210509225838772.png)

### 

### 发送有序广播

广播是一种跨进程进行通信的，前面发送的都是标准广播，下面尝试有序广播，有序广播的发送只需要将方法变为sendOrderedBroadcast即可，同时因为有序广播对于广播的接收者来说是有序的，可以在intent-filter标签中配置android:priority选项，用于配置优先级。

同时，前面也说过优先级高的广播收到广播后可以对这个广播进行截断，下面的案例则将演示隔断

第一个进程

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = findViewById(R.id.button1);
        button.setOnClickListener((View v) -> {
            Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST");
            //高版本加入此判断后本进程，跨进程都能发送接收到广播
            if(Build.VERSION.SDK_INT >= 26) {
                intent.addFlags(0x01000000);
            }
            Log.e("MainActivity","按钮点击");
            //有序广播
            sendOrderedBroadcast(intent, null);
        });
    }

}
```

MyBroadcastReceiver.java

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e("MyBroadcastReceiver", "广播接收");
        Toast.makeText(context, "Received in MyBroadcastReceiver", Toast.LENGTH_LONG).show();
        //进行截断
        abortBroadcast();
    }
}
```

```xml
<receiver android:name=".MyBroadcastReceiver"
          android:enabled="true"
          android:exported="true">
    <!--设置优先级，表示本进程收到广播的优先级-->
    <intent-filter android:priority="100">
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>

</receiver>
```

第二个进程

AnotherBroadcastReceiver.java，就是普通的接收广播的方法，没有什么特别的

```java
public class AnotherBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Log.e("AnotherBroadcastReceiver", "广播接收");
        Toast.makeText(context, "Received in AnotherBroadcastReceiver", Toast.LENGTH_LONG).show();
    }
}
```

```xml
<receiver android:name=".AnotherBroadcastReceiver"
          android:enabled="true"
          android:exported="true">
    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>

</receiver>
```

这个案例可以将进程一发送的广播，然后进程一和进程二可以收到，同时进程一设置了优先级为100，表示比进程二更早收到广播，当进程一收到广播之后，进行了广播阶段，所以进程二收不到广播



# 使用本地广播

前面定义的广播都是跨进程的属于全局系统广播，发出的广播可以被其他任何应用程序接收到，并且也可以接收到来自其他进程的广播，那么这里存在非常大的安全问题，Android可以支持发送本地广播，发送本地广播只有本地进程可以收到

需要注意的是本地广播的接收只能使用动态注册，因为静态注册就是为了让程序在未启动的时候也能接收到广播，而发送本地广播的时候应用程序肯定启动了，所以完全不需要使用静态注册的功能

```java
public class MainActivity extends AppCompatActivity {

    private LocalBroadcastManager localBroadcastManager;

    private IntentFilter intentFilter;

    private LocalReceiver localReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        localBroadcastManager = LocalBroadcastManager.getInstance(this);

        Button button = findViewById(R.id.button1);
        button.setOnClickListener((View v) -> {
            Intent intent = new Intent("com.example.broadcasttest.MY_BROADCAST_LOCAL");
            //高版本加入此判断后本进程，跨进程都能发送接收到广播
            if(Build.VERSION.SDK_INT >= 26) {
                intent.addFlags(0x01000000);
            }
            Log.e("MainActivity","按钮点击");
            localBroadcastManager.sendBroadcast(intent);
        });
        intentFilter = new IntentFilter("com.example.broadcasttest.MY_BROADCAST_LOCAL");
        localReceiver = new LocalReceiver();
        //注册本地广播器
        localBroadcastManager.registerReceiver(localReceiver, intentFilter);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        localBroadcastManager.unregisterReceiver(localReceiver);
    }

    class LocalReceiver extends BroadcastReceiver{

        @Override
        public void onReceive(Context context, Intent intent) {
            Log.e("LocalReceiver", "本地广播接收");
            Toast.makeText(context, "Received in BroadLocal", Toast.LENGTH_LONG).show();
        }
    }
}
```

