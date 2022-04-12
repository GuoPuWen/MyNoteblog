# Android多线程编程

线程的基本用法和Java中的多线程编程基本一致，只不过对于UI组件来说是不能这样使用的，因为UI组件是线程不安全的，例如以下例子

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Update"/>

    <TextView
        android:id="@+id/tx"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:layout_gravity="center_vertical" />

</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private TextView tx;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tx = findViewById(R.id.tx);
        Button btn1 = findViewById(R.id.btn1);
        btn1.setOnClickListener((View v) -> {
            new Thread(() -> {
                tx.setText("Hello Java");
            }).start();
        } );
    }
}
```

上面在子线程中尝试修改TextView中的值，运行程序发现程序崩溃了，同时报出下面的错误

![image-20210519165152953](http://cdn.noteblogs.cn/image-20210519165152953.png)

那如果需要有非常耗时的操作然后更新UI组件呢？Android中提供了一套异步消息处理机制，完美的解决在子线程中更新UI的操作

### 异步消息处理Handle

```java
public class MainActivity extends AppCompatActivity {

    public static final int UPDATE_TEXT = 1;

    private TextView tx;

    private Handler handler = new Handler(){
        @Override
        public void handleMessage(@NonNull Message msg) {
            super.handleMessage(msg);
            switch (msg.what){
                case UPDATE_TEXT:
                    tx.setText("Hello Java");
                    break;
                default:break;
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tx = findViewById(R.id.tx);
        Button btn1 = findViewById(R.id.btn1);
        btn1.setOnClickListener((View v) -> {
            new Thread(() -> {
                Message message = new Message();
                message.what = UPDATE_TEXT;
                handler.sendMessage(message);
            }).start();
        } );
    }
}
```

下面解析异步消息处理机制，Android中异步消息处理主要由4部分组成：Message、Handler、MessageQuene、Looper

- Message：在线程之间传递消息，可以在内部携带少量的信息，用于在不同的线程之间交换数据，上面使用的是what字段，还可以使用arg1和arg2，或者obj字段
- Handler：处理器，用于发送消息和处理消息的，发送消息一般是Handler的sendMessage()方法，而发出的消息经过一系列的处理之后，最终会传递到Handler的handlerMessage()方法
- MessageQuene：消息队列，主要用于存放所有通过Handler发送的消息，这部分消息会一直存在于消息队列中，等待被处理，每个线程只有一个MessageQuene对象
- Looper：Looper是每个线程中的MessageLoop的管家，调用Looper的loop方法之后，就会进入到一个无限循环中，然后每当发现MessageQuene中存在一条消息，就会将它取出，并传递到Handler的handleMessage()方法中，同样每个线程也只会有一个Looper对象

异步消息处理的流程是：

首先在主线程中创建一个Handler对象，并重写handleMessage()方法，然后当子线程中需要进行UI操作时，就会创建一个Message对象，并通过Handler将这条消息发送出去，之后这条消息会被添加到MessageQuene的队列中等待被处理，而Looper则会一直尝试从MessageQuene中取出待处理的消息，最后分发回Handler的handleMessage()方法中，由于Handler是在主线程中创建的，所以这个时候handleMessage()方法也是在主线程中执行的，所以可以执行UI操作![image-20210519230337867](http://cdn.noteblogs.cn/image-20210519230337867.png)

### 使用AsyncTask

为了更方便的在子线程中操作UI，Android还提供了更好的工具AsyncTask，AsyncTask是一个抽象类，需要一个子类去继承它，在继承的时候可以指定三个泛型参数：

- Params：执行AsyncTask时需要传入的参数，可用于后台任务使用
- Progress：后台执行任务时，如果需要在界面上显示当前的进度，可以使用这里的泛型作为单位
- Result：任务执行完毕之后，如果需要对结果进行返回，可以这里指定泛型作为返回类型

继承AsyncTask一般需要重写以下方法：

1. onPreExecute：这个方法会在后台任务开始执行之前调用，用于一些界面上的初始化操作
2. doInBackground(Params..)：这个方法中的代码都会在子线程中进行，应该在这里处理耗时的任务，任务一旦完成可以过return语句来将执行结果进行返回，如果要更新UI元素，可以调用publishProgress()来完成
3. onProgressUpdate(Progress..)：当在后台调用了publishProgress之后，onProgressUpdate就很快会被调用，在这个方法中可以对UI进行操作
4. onPostExecute()：当后台任务执行完毕并通过return语句进行返回时，这个方法会被调用

# 服务的基本用法

首先定义两个按钮

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Start Service"
        />

    <Button
        android:id="@+id/btn2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Stop Service"/>

</LinearLayout>
```

创建一个类MyService继承Service类

```java
public class MyService extends Service {

    public static final String TAG = "MyService";

    public MyService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e(TAG, "onCreate");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e(TAG, "onDestroy");
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e(TAG, "onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }
}
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn1 = findViewById(R.id.btn1);
        Button btn2 = findViewById(R.id.btn2);
        btn1.setOnClickListener((View view) -> {
            Intent intent = new Intent(this, MyService.class);
            startService(intent);
        });

        btn2.setOnClickListener((View v) -> {
            Intent intent = new Intent(this, MyService.class);
            stopService(intent);
        });
    }
}
```

使用Intent进行一个服务的开启与关闭同时在服务的对应生命周期函数中打印了日志

![image-20210520135816818](http://cdn.noteblogs.cn/image-20210520135816818.png)

### 活动与服务进行通信



