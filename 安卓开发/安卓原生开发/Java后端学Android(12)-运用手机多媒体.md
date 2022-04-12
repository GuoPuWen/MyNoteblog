# 使用通知

通知是指当应用程序不再在前台运行的时候，可以向用户发出一些重要的提示信息，发出通知后，手机最上方的状态栏会显示一个通知的图标，下拉状态栏可以看到详细内容

### 通知的基本用法

首先需要一个NotificationManager来对通知进行管理，调用context的getSystemService方法可以获取，接下来需要一个Builder构造器来构造一个Notification对象，Android8.0开始，废弃了Builder(@NonNull Context context)方法，改用Builder(@NonNull Context context, @NonNull String channelId)，也就是说需要传入一个channelId

从Android8开始，谷歌引入了NotificationChannel（通知渠道）这个概念，也就是说每一个通知需要对应一个通知渠道，每个app可以自由的创建当前app拥有的通知渠道，用户可以自由的选择是否关闭某种通知渠道的通知，例如对于支付宝我只是需要收款的通知，而不需要其他乱七八糟的通知，这样对用户的体验就更好了

Notification的创建时一种类似于Stream流的使用方式在最终的build()方法之前可以使用一系列的方法对Notification进行丰富，这是一种链式编程的思想，例如：

```java
Notification notification = new NotificationCompat.Builder(this, "chat")
    .setAutoCancel(true)
    .setContentTitle("收到聊天消息")
    .setContentText("今天晚上吃什么")
    .setWhen(System.currentTimeMillis())
    .setSmallIcon(R.mipmap.ic_launcher)
    .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
    //在build()方法之前还可以添加其他方法
    .build();
```

接着只需要调用NotificationManager的notify方法即可，接收两个参数，一个id确保每一个通知的id都不一样，第二个便是Notification对象

下面是一个完整的实例：

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">



    <Button
        android:id="@+id/send_notice"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>


</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @RequiresApi(api = Build.VERSION_CODES.O)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //创建通知渠道
        NotificationChannel channel = new NotificationChannel("chat", "测试通知", NotificationManager.IMPORTANCE_DEFAULT);

        Button btn = findViewById(R.id.send_notice);
        btn.setOnClickListener((View v) -> {
            NotificationManager manager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
            Notification notification = new NotificationCompat.Builder(this, "chat")
                    .setAutoCancel(true)
                    .setContentTitle("收到聊天消息")
                    .setContentText("今天晚上吃什么")
                    .setWhen(System.currentTimeMillis())
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                    //在build()方法之前还可以添加其他方法
                    .build();
            manager.createNotificationChannel(channel);
            manager.notify(1, notification);
        });
    }
}
```

RequiresApi注解用于标注在表示应仅在给定的API级别或更高级别上调用带注释的元素，这里表示在26版本之上调用

运行程序，点击按钮则模拟器的状态栏出现一则通知

![image-20210515213429707](http://cdn.noteblogs.cn/image-20210515213429707.png)

### 使用点击

下面将会做状态栏上的通知能够进行点击事件，能够启动另外一个活动。只需要使用PendingIntent即可做到，根据字面的意思PendingIntent是延迟的Intent

```java
public class MainActivity extends AppCompatActivity {


    @RequiresApi(api = Build.VERSION_CODES.O)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //创建通知渠道
        NotificationChannel channel = new NotificationChannel("chat", "测试通知", NotificationManager.IMPORTANCE_DEFAULT);

        Button btn = findViewById(R.id.send_notice);
        btn.setOnClickListener((View v) -> {

            Intent intent = new Intent(this, NotificationActivity.class);
            PendingIntent pi = PendingIntent.getActivity(this, 0, intent, 0);

            NotificationManager manager = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);
            Notification notification = new NotificationCompat.Builder(this, "chat")
                    .setAutoCancel(true)
                    .setContentTitle("收到聊天消息")
                    .setContentText("今天晚上吃什么")
                    .setWhen(System.currentTimeMillis())
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher))
                    .setContentIntent(pi)
                    //在build()方法之前还可以添加其他方法
                    .build();
            manager.createNotificationChannel(channel);
            manager.notify(1, notification);
//            startActivity(intent);
        });
    }
}
```

Builder构造器还存在一系列的方法，用于控制通知的效果

| 方法                                                 | 作用                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| setAutoCancel(boolean boolean)                       | 设置点击通知后自动清除通知                                   |
| setContent(RemoteView view)                          | 设置自定义通知                                               |
| setContentTitle(String string)                       | 设置通知的标题内容                                           |
| setContentText(String string)                        | 设置通知的正文内容                                           |
| setContentIntent(PendingIntent intent)               | 设置点击通知后的跳转意图                                     |
| setWhen(long when)                                   | 设置通知被创建的时间                                         |
| setSmallIcon(int icon)                               | 设置通知的小图标注意：只能使用纯alpha图层的图片进行设置，小图标会显示在系统状态栏上 |
| setLargeIcon(Bitmap icon)                            | 设置通知的大图标 下拉系统状态栏时就能看见                    |
| setPriority(int pri)                                 | 设置通知的重要程度                                           |
| setStyle(Style style)                                | 设置通知的样式 比如设置长文字、大图片等等                    |
| setVisibility(int defaults)                          | 设置默认                                                     |
| setLight(int argb, int onMs, int offMs)              | 设置呼吸闪烁效果                                             |
| setSound(Uri sound)                                  | 设置通知音效                                                 |
| setVibrate(long[] pattern)                           | 设置震动效果，数组包含手机静止时长和震动时长<br />下标0代表手机静止时长 <br />下标1代表手机整的时长<br />下标3，4，5.......以此类推<br />还需要在AndroidManifest.xml中声明权限：<br /> |
| setColor(int argb)                                   | 设置通知栏颜色                                               |
| setCategory(String category)                         | 设置通知类别                                                 |
| setFullScreenIntent(PendingIntent intent, boolean b) | 设置弹窗显示                                                 |



# 调用系统摄像头和相册

​	

