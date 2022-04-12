本系列为《Android 第一行代码第二版》读书笔记

# 一、使用Intent在活动之间穿梭

上篇文章使用AS的Empty Activity体验了Hello World，也就是说对于Android来说是Actiity叠加的，可以使用多个Activity，那么新建一个Empty 的Activity并创建其对应的layout文件。

现在有两个Activity分别为MainActivity、SecondActivity。现在需要完成的功能是在MainActivity上有一个按钮，点击这个按钮可以进入到SecondActivity，SecondActivity上也有一个按钮

上面的需求就是要进行两个Activity之间的切换，那么很必然的在切换的同时，需要携带数据两个Activity之间如何传递数据？那么就需要使用到Intent

### 1.1 显式Intent

环境准备

first_layout.xml：定义一个Button_1

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/button_1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="button 1"></Button>

</LinearLayout>
```

second_layout.xml：定义一个Button_3

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".SecondActivity">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/button_3"
        android:text="Button 3"></Button>

</LinearLayout>
```

使用显式的intent可以从一个Activity调到另外一个Activity

例如在MainActivity中定义按钮点击事件

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.setContentView(R.layout.first_layout);
        Button button1 = (Button)this.findViewById(R.id.button_1);
        button1.setOnClickListener((View view) -> {
            //显式Intent
            Intent intent = new Intent(this, SecondActivity.class);
            this.startActivity(intent);
        });

    }

}
```

```java
public Intent(Context packageContext, Class<?> cls) {
    mComponent = new ComponentName(packageContext, cls);
}
```

Intent函数接收两个参数：

- 第一个参数Context为上下文，一般使用this即可
- 第二个参数为要启动的目标Activity

然后使用startActivity便可启动该Activity

### 1.2 使用隐式Intent

使用隐式Intent则并不明确要启动哪一个活动，而是在活动中指定一些action和category信息，然后系统会去分析这个Intent，并找到合适的活动进行启动

还是上面的例子，在AndroidManifest.xml定义一些信息

```xml
<activity android:name=".SecondActivity">
    <intent-filter>
        <action android:name="com.example.activitytest.ACTION_START" />
		<!--默认的category --> 
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="com.example.activitytest.MY_CATEGORY" />
    </intent-filter>
</activity>
```

接着在MainActivity中

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.setContentView(R.layout.first_layout);
        Button button1 = (Button)this.findViewById(R.id.button_1);
        button1.setOnClickListener((View view) -> {
            //使用隐式Intent
            Intent intent = new Intent("com.example.activitytest.ACTION_START");      		        
        intent.addCategory("com.example.activitytest.MY_CATEGORY");
        });
    }
}
```

action标签指定当前活动可以响应com.example.activitytest.ACTION_START这个活动，而category标签包含一些附加信息，更精确的指明当前的活动能够响应Intent中还可以带有的category，只有在action和category中的内容同时匹配上Intent的内容的时候，该活动才可以响应Intent

### 1.3 向下一个活动传递数据

两个Activity之间需要进行关联需要Intent，所以传递数据也是使用Intent：

- Intent.putExtra(String name, @Nullable String value) ：向Intent中写入数据
- Intent.getxxxExtra：向Intent中取得数据

当然这些方法都是一些重载的方法，用于传输不同额数据

例如现在MainActivity需要向SecondActivity传递数据

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.setContentView(R.layout.first_layout);
        Button button1 = (Button)this.findViewById(R.id.button_1);
        button1.setOnClickListener((View view) -> {
            //显式Intent
            String data = "Hello SecondActivity";
            Intent intent = new Intent(this, SecondActivity.class);
            intent.putExtra("data", data);
            this.startActivity(intent);
        });
    }
}
```

SecondActivity.java

```java
public class SecondActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second_layout);
        //获取数据
        Intent intent = this.getIntent();
        String data = intent.getStringExtra("data");
        Log.d("Intent的数据" , data);
        //通知
        Toast.makeText(this,data, Toast.LENGTH_SHORT).show();
    }
}
```

### 1.4 向上一个活动传递数据

既然可以向下传递一个数据，那么肯定可以向上传递数据，在Activity中有一个startActivityForResult方法用于启动活动，但是这个方法期望在活动销毁的时候能够返回一个结果给上一个活动。startActivityForResult方法接收两个参数

- Intent：启动的Activity
- requestCode：用于在回调之后判断数据的来源

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.setContentView(R.layout.first_layout);
        Button button1 = (Button)this.findViewById(R.id.button_1);
        button1.setOnClickListener((View view) -> {
            Intent intent = new Intent(this, SecondActivity.class);
            this.startActivityForResult(intent, 1);
        });
    }
    
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        //super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case 1:
                if (resultCode == RESULT_OK) {
                    String dataReturn = data.getStringExtra("data_return");
                    Toast.makeText(this, dataReturn, Toast.LENGTH_SHORT).show();
                    Log.d("向上传递数据", dataReturn);
                }
                break;
            default:
        }
    }
}
```

SecondActivity.java

```java

public class SecondActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.second_layout);
        //向上传递数据
        Button button = (Button) this.findViewById(R.id.button_3);
        button.setOnClickListener((View v) -> {
            Intent intent = new Intent();
            intent.putExtra("data_return", "Hello Return");
            //这个方法很关键，用于向上传递数据，第一个参数为返回处理结果一般使用RESULT_OK或者RESULT_CANCELED
            this.setResult(RESULT_OK, intent);
            //手动销毁Activity
            this.finish();
        });
    }
}
```

同时，在SecondActivity活动销毁的时候还会去调用上一个活动的onActivityResult方法在这个方法中可以获取到销毁活动的数据，该方法引入三个参数：

- requestCode：就是startActivityForResult中传入的int值
- resultCode：返回结果
- data：销毁的intent

同时，上面通过finish()方法来销毁活动的，如果使用回退键也可以销毁活动，所以可以在活动的生命周期函数上定义

SecondActivity.java

```java
    @Override
    public void onBackPressed() {
        Intent intent = new Intent();
        intent.putExtra("data_return", "Hello Return");
        this.setResult(RESULT_OK, intent);
        this.finish();
    }
```

