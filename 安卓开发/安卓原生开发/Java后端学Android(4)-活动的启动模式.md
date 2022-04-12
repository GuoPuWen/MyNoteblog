# 活动的启动模式

在Java应用程序中一个Bean有多例的，有单例的。而在Android中的活动也有多个启动模式，一共有四种standard、singleTop、singleTask、singleInstance。下面依次介绍

### standard

standard是活动默认的启动模式，在不进行显式指定的情况下，都使用这种模式。

在standard模式下，每当启动一个新的活动，它就会在返回栈中入栈，并处于栈顶的位置，系统不会在乎这个活动是否已经在返回栈中存在，每次创建该活动都会启动一个新的实例。

例如下面一个实例，在MainActivity的基础上创建MainActivity，查看打印结果

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.setContentView(R.layout.first_layout);
    Button button1 = (Button)this.findViewById(R.id.button_1);
    Log.d("FirstActivity", this.toString());
    button1.setOnClickListener((View v) -> {
        Intent intent = new Intent(this, MainActivity.class);
        this.startActivity(intent);
    });
}
```

![image-20210502132133773](http://cdn.noteblogs.cn/image-20210502132133773.png)

查看打印日志，发现每次创建的活动都不一样

standard模式下的启动示意图为

![image-20210502132227418](http://cdn.noteblogs.cn/image-20210502132227418.png)

### singleTop

standard模式下的无论活动是否已经在栈顶都会不断的创建新活动，这样很明显浪费内存空间，而使用singleTop模式，当活动的启动模式指定为singleTop，在启动活动时如果发现返回栈的栈顶已经是该活动，则认为可以直接使用它，不会在创建新的活动

还是刚才的例子，只是将启动模式改为singleTop，修改AndroidManifest.xml文件

```xml
<activity
          android:name=".MainActivity"
          android:label="This is first Activity"
          android:launchMode="singleTop">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

将模式改为singleTop后，不论点击多少次按钮日志输出的hashcode值都只有一个，因为该活动已经在栈顶了，singleTop模式下不会为该已经在栈顶的活动创建新的活动

==但是，如果当MainActivity并未处于栈顶位置时，再次启动MainActivity时，还是会创建新的实例的==

### singleTask

singleTop模式下并没有改变一个实例下存在多个活动的问题，因为假如该活动不处于栈顶，那么还是会创建该活动的实例的，而使用singleTask则可以解决重复创建活动的问题

使用singleTask，每次启动该活动的时候系统首先会在返回栈中检查是否存在该活动的实例，如果发现已经存在则直接使用该实例，并把在这个活动之上的所有活动都出栈，如果没有发现则创建一个新的活动实例

修改上面的例子，在MainActivity的Button上启动SecondActivity，然后SecondActivity的Button上回到MainActivity，同时重写MainActivity的onRestart方法和SecondActivity的onDestory方法

MainActivity.java

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.setContentView(R.layout.first_layout);
    Button button1 = (Button)this.findViewById(R.id.button_1);
    Log.d("FirstActivity", this.toString());
    button1.setOnClickListener((View v) -> {
        Intent intent = new Intent(this, SecondActivity.class);
        this.startActivity(intent);
    });

}
@Override
protected void onRestart() {
    super.onRestart();
    Log.d("FirstActivity", "onRestart");
}
```

SecondActivity.java

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.second_layout);
    Button button = (Button) this.findViewById(R.id.button_3);
    button.setOnClickListener((View v) -> {
        Intent intent = new Intent(this, MainActivity.class);
        startActivity(intent);

    });
}
@Override
protected void onDestroy() {
    super.onDestroy();
    Log.d("SecondActivity", "onDestroy");
}
```

AndroidManifest.xml

```xml
<activity
          android:name=".MainActivity"
          android:label="This is first Activity"
          android:launchMode="singleTask">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

查看打印日志，可以发现执行了SecondActivity的destory方法

![image-20210502152440592](http://cdn.noteblogs.cn/image-20210502152440592.png)

也就是说使用了singleTask模式的活动，当需要使用时会查找活动栈中是否有该活动，如果有该活动，那么将该活动上面的所有活动都出栈。例如上面的例子当SecondActivity点击button使用MainActivity的时候，活动栈的顺序为

![image-20210502152901439](http://cdn.noteblogs.cn/image-20210502152901439.png)

因为MainActivity是singleTask模式的，所以会在该返回栈中查找是否存在MainActivity，发现是存在的，所以将MainActivity上面的所有活动都出栈，那么即会调用它的destory方法

### singleInstance

指定为singleInstance模式的活动会启用一个新的返回栈来管理这个活动，这样做的好处是可以做到不同应用程序之间的共享问题，假如程序中有一个活动是允许其他程序调用的，也就是共享这个活动的实例，而使用singleInstance因为重新启用了一个新的返回栈所以可以做到共享活动

AndroidManifest.xml

```xml
<activity android:name=".ThirdActivity">
</activity>
<activity android:name=".SecondActivity" android:launchMode="singleInstance">
    <intent-filter>
        <action android:name="com.example.activitytest.ACTION_START" />

        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="com.example.activitytest.MY_CATEGORY" />
    </intent-filter>
</activity>
<activity
          android:name=".MainActivity"
          android:label="This is first Activity"
          android:launchMode="standard">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

MainActivity.java

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.setContentView(R.layout.first_layout);
    Button button1 = (Button)this.findViewById(R.id.button_1);
    Log.d("FirstActivity", this.toString());
    Log.d("FirstActivity", "Task id is" + this.getTaskId());
    button1.setOnClickListener((View v) -> {
        Intent intent = new Intent(this, SecondActivity.class);
        this.startActivity(intent);
    });

}
```

SecondActivity.java

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.second_layout);
    Log.d("SecondActivity", "Task id is" + this.getTaskId());
    Button button = (Button) this.findViewById(R.id.button_3);
    button.setOnClickListener((View v) -> {
        Intent intent = new Intent(this, ThirdActivity.class);
        startActivity(intent);
    });
}
```

ThirdActivity.java

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.third_layout);
    Log.d("ThirdActivity", "Task id is" + this.getTaskId());
}
```

上述程序打印该属于本活动的返回栈的Task id号，通过查看日志，可以发现

![image-20210502154323504](http://cdn.noteblogs.cn/image-20210502154323504.png)

由于SecondActivity指定了启动模式为singleInstance所以创建时会重新启动一个返回栈存放该活动

自然在ThirdActivity中按back键，那么会直接返回到MainActivity，然后在MainActivity按下返回键，会回到SecondActivity，最后在SecondActivity中按下返回键最后退出程序

![image-20210502154930192](http://cdn.noteblogs.cn/image-20210502154930192.png)

### 总结

- standard：默认的启动方式，每次启动一个活动都会重新创建
- singleTop：如果改活动处于栈顶，则不会创建新活动，不处于栈顶则创建新活动
- singleTask：如果返回栈中存在该活动，那么将该活动之上的所有活动统统出栈，将该活动置于栈顶，如果不存在该活动则创建
- singleInstance：会重新启用一个新的返回栈来创建该活动，通常用于共享活动的实例