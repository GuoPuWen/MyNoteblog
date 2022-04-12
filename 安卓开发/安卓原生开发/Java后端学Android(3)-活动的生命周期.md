# 一、活动的生命周期

Android中的活动是可以叠加的，这也意味着每一个活动都应该具有对应的生命周期，前面每次使用的都是onCreate方法，这也是生命周期函数里面的重要方法

掌管每一个活动的叫做活动栈，很显然一个活动能叠加到另外一个活动上，需要栈这种先进后出的数据结构。每当启动一个新的活动，它就会在返回栈上入栈，系统总会显示处于栈顶的活动给用户

### 1.1 活动状态

- 运行状态：当一个活动处于返回栈的栈顶，活动处于运行状态
- 暂停状态：当一个活动不在处于栈顶的时候，但仍然是可见的，这种状态的原因是并不是每一个活动都需要占满一个屏幕，例如说一些对话框活动
- 停止状态：当一个活动不在处于栈顶位置，并且完全不可见的时候，处于停止状态，系统仍然会为这种活动保存相应的状态和变量，但是当系统其它地方需要内存的时候，处于停止状态的活动会被系统回收
- 销毁状态：当一个活动从返回栈中移除之后，就处于销毁状态

### 1.2 活动的生存期

Activity 类中定义了七个回调方法，覆盖了活动生命周期的每一个环节

- onCreate()：这个方法你已经看到过很多次了，每个活动中我们都重写了这个方法，它会在活动第一次被创建的时候调用。你应该在这个方法中完成活动的初始化操作，比如说加载布局、绑定事件等。

- onStart()：这个方法在活动由不可见变为可见的时候调用。
-  onResume()：这个方法在活动准备好和用户进行交互的时候调用。此时的活动一定位于返回栈的栈顶，并且处于运行状态。
-  onPause()：这个方法在系统准备去启动或者恢复另一个活动的时候调用。我们通常会在这个方 法中将一些消耗 CPU 的资源释放掉，以及保存一些关键数据，但这个方法的执行速度 一定要快，不然会影响到新的栈顶活动的使用。

- onStop()：这个方法在活动完全不可见的时候调用。它和 onPause()方法的主要区别在于，如果启动的新活动是一个对话框式的活动，那么 onPause()方法会得到执行，而 onStop()方法并不会执行。

- onDestroy()：这个方法在活动被销毁之前调用，之后活动的状态将变为销毁状态。

- onRestart()：这个方法在活动由停止状态变为运行状态之前调用，也就是活动被重新启动了。 以上七个方法中除了 onRestart()方法，其他都是两两相对的，从而又可以将活动分为三种生存期。

==完整生存期==

活动在 onCreate()方法和 onDestroy()方法之间所经历的，就是完整生存期。一般情况下，一个活动会在 onCreate()方法中完成各种初始化操作，而在 onDestroy()方法中完成释放内的操作。

==可见生存期==

活动在 onStart()方法和 onStop()方法之间所经历的，就是可见生存期。在可见生存期内，活动对于用户总是可见的，即便有可能无法和用户进行交互。我们可以通过这两 个方法，合理地管理那些对用户可见的资源。比如在 onStart()方法中对资源进行加载， 而在 onStop()方法中对资源进行释放，从而保证处于停止状态的活动不会占用过多内存。

==前台生存期==

活动在 onResume()方法和 onPause()方法之间所经历的，就是前台生存期。在前台生存期内，活动总是处于运行状态的，此时的活动是可以和用户进行相互的，我们平时看到和接触最多的也这个状态下的活动。

### 1.4 体验生命周期

环境搭建：三个Activity：MainActivity、NormalActivity、DialogActivity见名之意，主活动、正常活动(覆盖整个屏幕的活动)、对话框形式的活动(不需要暂满整个屏幕的活动)。三个layout文件

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.d("创建方法", "onCreate");

        Button normalButton = (Button) this.findViewById(R.id.button_1);
        normalButton.setOnClickListener((View v) -> {
            Intent intent = new Intent(this, NormalActivity.class);
            this.startActivity(intent);
        });
        Button dialogButton = (Button) this.findViewById(R.id.button_2);
        dialogButton.setOnClickListener((View v) -> {
            Intent intent = new Intent(this, DialogActivity.class);
            this.startActivity(intent);
        });
    }

    @Override
    protected void onStart() {
        super.onStart();
        Log.d("启动方法", "OnStart");
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.d("准备交互", "OnResume");
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.d("活动启动另外一个活动", "OnPause");
    }

    @Override
    protected void onStop() {
        super.onStop();
        Log.d("活动暂停", "OnStop");
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d("活动销毁", "onDestroy");
    }
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical">
    
    <Button
        android:id="@+id/button_1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Normal Activity">
    </Button>

    <Button
        android:id="@+id/button_2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Dialog Activity">
    </Button>

</LinearLayout>
```

NormalActivity.java

```java
public class NormalActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.normal_layout);
    }
}
```

normal_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".NormalActivity">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="This is normal activity"></TextView>

</LinearLayout>
```

DialogActivity.java

```java
public class DialogActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.dialog_layout);
        AlertDialog.Builder dialog = new AlertDialog.Builder(this);
        dialog.setTitle("This is Dialog");
        dialog.setMessage("Something id Dialog");
    }
}
```

dialog_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".DialogActivity">

</LinearLayout>
```

对照生命周期图

![img](http://cdn.noteblogs.cn/15207-20151230134402026-2097191680.jpg)

运行AS，首先到达MainActivity，自然是运行其onCreate()、OnStart()、OnResume()方法，可以看logcat日志

![image-20210502101652902](http://cdn.noteblogs.cn/image-20210502101652902.png)

当点击Normal Activity按钮时，也就是要切换到另外一个活动了但是这个活动是需要暂满整个屏幕的活动，所以是可见生存期内的活动

![image-20210502101953407](http://cdn.noteblogs.cn/image-20210502101953407.png)

当点击back按钮时，返回MainActivity

![image-20210502102248065](Java后端学Android(3)-活动的生命周期、活动的启动模式.assets/image-20210502102248065.png)

也就是说对于MainActivity启动NormalActivity最后又回到MainActivity活动，经历过的方法为：

OnCreate -> OnStart -> OnResume -> OnPause -> OnStop -> OnRestart -> OnStart -> OnResume

接着实验MainActivity启动DialogActivity，最后又回到MainActivity活动

![image-20210502102547163](http://cdn.noteblogs.cn/image-20210502102547163.png)

back键返回按照流程图中箭头指向运行方法

也就是说MainActivity启动DialogActivity，最后又回到MainActivity活动，经历过的方法为

OnCreate -> OnStart -> OnResume -> OnPause -> OnResume 

最后退出整个程序执行OnDestroy方法

### 1.5 活动被回收怎么办

A活动启动B活动，假如此时内存空间不足，系统将A活动进行回收了，当我们(此时在B活动)点击back时，系统重新运行onCreate方法执行一整套生命周期。但是如果在A中有一些重要的临时数据，但是因为被系统强制回收了，这些数据自然也丢失了，这会很影响系统运行或者用户的体验

可以使用onSaveInstanceState方法，可以看到里面绑定一个Bundle变量，没错就是将

```java
protected void onSaveInstanceState(@NonNull Bundle outState)
```

一些重要的临时数据保存在Bundle中，然后在onCreate方法中取出，因为onCreate方法也存在一个Bundle变量，可用于传递数据。

到目前为止已经学习了使用Intent在不同的活动上传递数据，使用Bundle在不同的生命周期之间传递数据，当然也可以结合使用