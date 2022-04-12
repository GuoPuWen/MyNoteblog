# Fragment

Fragment叫做碎片，是一种可以嵌入在活动中的UI片段，可以充分利用大屏幕的空间，在平板应用上应用的比较广泛，Fragment的定义为小活动，也就是Fragment是比一个活动更细化的管理空间，可以理解为小活动，因为它同样具有生命周期

最简单的使用Fragment的例子是在平板应用上，左边一栏为新闻的标题，而右边对应着每一个标题的内容，可以将这两栏的内容放置在不同的Fragment上。使用Fragment可以很好的进行模块化管理，当然Fragment必须依赖于活动才能存在，并且Fragment具有自己的生命周期，并且能在活动运行期间动态的添加和修改Fragment

### 1.简单使用

left_fragment.xml

```xml
<LinearLayout android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android">


    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="Button"/>

</LinearLayout>
```

right_fragment.xml

```xml
<LinearLayout android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#00ff00"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android">


    <TextView
        android:id="@+id/tx1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:text="This is right fragment"/>

</LinearLayout>
```

LeftFragment.java

```java
public class LeftFragment extends Fragment {


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.left_fragment, container, false);
    }
}
```

RightFragment.java

```java
public class RightFragment extends Fragment {



    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.right_fragment, container, false);
    }
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/left_fragment"
        android:name="com.example.fragmenttask.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"/>

    <fragment
        android:id="@+id/right_fragment"
        android:name="com.example.fragmenttask.RightFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"/>


</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

这样就完成了一个简单的Fragment的使用，建立两个fragment.xml分别与对应的Fragment类对应，然后在主布局中使用。将模拟器改为平板，运行结果如下

![image-20210507200711451](http://cdn.noteblogs.cn/image-20210507200711451.png)

### 2. 动态添加

Fragment碎片的真正强大的地方在于可以在程序运行过程中动态的添加大活动中，下面将做一个点击按钮切换右边碎片的案例

新建一个another_right_fragment.xml

```xml
<LinearLayout android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#ffff00"
    xmlns:android="http://schemas.android.com/apk/res/android">


    <TextView
        android:id="@+id/tx2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="20sp"
        android:text="This is another right fragment"/>
</LinearLayout>
```

同样的，也要创建一个AnotherRightFragment.java类用来解析这个碎片

```java
public class AnotherRightFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.another_right_fragment, container, false);
    }
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/left_fragment"
        android:name="com.example.fragmenttask.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"/>

<!--    <fragment-->
<!--        android:id="@+id/right_fragment"-->
<!--        android:name="com.example.fragmenttask.RightFragment"-->
<!--        android:layout_width="0dp"-->
<!--        android:layout_height="match_parent"-->
<!--        android:layout_weight="1"/>-->
    <FrameLayout
        android:id="@+id/right_fragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"/>


</LinearLayout>
```

```java
public class MainActivity extends AppCompatActivity  implements View.OnClickListener {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = findViewById(R.id.button1);
        button.setOnClickListener(this::onClick);
        replaceFragment(new RightFragment());
    }


    private void replaceFragment(Fragment fragment){
        //获取碎片管理器
        FragmentManager supportFragmentManager = getSupportFragmentManager();
        //获取事务
        FragmentTransaction fragmentTransaction = supportFragmentManager.beginTransaction();
        //动态替换
        fragmentTransaction.replace(R.id.right_fragment, fragment);
        //提交事务
        fragmentTransaction.commit();
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.button1:
                replaceFragment(new AnotherRightFragment());
                break;
        }
    }
}
```

上面的案例点击按钮可以将右边的Fragment进行切换，但是会发现直接点击back会退出程序，可以使用fragmentTransaction.replace(R.id.right_fragment, fragment);方法进行返回栈，这样切换一次碎片，就会在栈中入栈，然后接着back将会依次出栈

```java

    private void replaceFragment(Fragment fragment){
        FragmentManager supportFragmentManager = getSupportFragmentManager();
        FragmentTransaction fragmentTransaction = supportFragmentManager.beginTransaction();
        fragmentTransaction.replace(R.id.right_fragment, fragment);
        //返回栈
        fragmentTransaction.addToBackStack(null);
        fragmentTransaction.commit();
    }
```





### 3. 碎片的生命周期

碎片的生命周期和活动的生命周期是非常相似的，在活动的生命周期中一共有四种状态：运行、暂停、停止、销毁。而在碎片的生命周期中也有这四种状态：

- 运行状态：当一个碎片是可见的，并且所关联的活动也是正在处于运行状态的时候，该碎片处于运行状态
- 暂停状态：当一个活动处于暂停状态时，与它相关联的可见碎片也处于暂停状态
- 停止状态：当一个活动处于停止状态时，与它相关联的碎片也会进入到停止状态，或者调用了FragmentTransaction的remove()、replace()方法将碎片从活动中移除，并且在事务提交之前调用addToBackStack()方法，这时的碎片也会进入到停止状态，进入到停止状态对用户来说是完全不可见的，也有可能会被系统回收
- 销毁状态：当活动被销毁时，与它相关联的碎片也会进入到销毁状态，或者调用了FragmentTransaction的remove()、replace()方法将碎片从活动中移除，并且在事务提交之前==没有==调用addToBackStack()方法，碎片也会进入到销毁状态

对于生命周期的方法，除了和活动的生命周期一样的方法之外，碎片还额外提供了：

- onAttach()：当碎片和活动建立关联的时候调用
- onCreateView()：为碎片创建视图（加载布局）时调用
- onActivityCreated()：确保与碎片相关联的活动一定已经创建完毕的时候调用

- onDestroyView()：当与碎片关联的视图被移除的时候调用

- onDetach()：当碎片和活动解除关联的时候调用

![img](http://cdn.noteblogs.cn/15207-20160127105914801-1347495601.jpg)

体验碎片的生命周期，在RightFragment.java中进行修改

```java
public class RightFragment extends Fragment {

    private static final String TAG = "RightFragment";

    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);
        Log.e(TAG, "onAttach");
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.e(TAG, "onCreate");
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        Log.e(TAG, "onCreateView");
        return inflater.inflate(R.layout.right_fragment, container, false);
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        Log.e(TAG, "onActivityCreated");
    }

    @Override
    public void onStart() {
        super.onStart();
        Log.e(TAG, "onStart");
    }

    @Override
    public void onResume() {
        super.onResume();
        Log.e(TAG, "onResume");
    }

    @Override
    public void onPause() {
        super.onPause();
        Log.e(TAG, "onPause");
    }

    @Override
    public void onStop() {
        super.onStop();
        Log.e(TAG,"onStop");
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.e(TAG, "onDestroyView");
    }

    @Override
    public void onDetach() {
        super.onDetach();
        Log.e(TAG, "onDetach");
    }
}
```

首先是RightFragment被加载，处于运行状态，调用onAttach、onCreate、onCreateView、onActivityCreated、onStart、onResume

![image-20210508145358575](http://cdn.noteblogs.cn/image-20210508145358575.png)

点击按钮，RightFragment被替换掉，因为加入了返回栈中，所以调用onPause、onStop、onDestoryView方法

![image-20210508145532486](http://cdn.noteblogs.cn/image-20210508145532486.png)

点击back，RightFragment会忠重新返回屏幕，调用onCreateView、onActivityCreated、onStart、onResume方法

![image-20210508145620319](http://cdn.noteblogs.cn/image-20210508145620319.png)