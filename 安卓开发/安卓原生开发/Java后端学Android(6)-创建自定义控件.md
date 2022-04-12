# 创建自定义控件

可以发现，Android中的一些常用的控件和一些基本布局都有以下的继承结构

![image-20210503101353907](http://cdn.noteblogs.cn/image-20210503101353907.png)

也就是直接或间接的继承View的，所有的布局都是直接或间接继承ViewGroup的。View是Android中的基本组件，能够在屏幕上绘制一块矩形区域，并能够响应这块区域的所有事件，因此使用的各种控件其实就是在View的基础上又添加了各自特有的功能，而ViewGroup是一种特殊的View，可以包含很多的子View和子ViewGroup，是一个放置控件和布局的容器

下面我们将做一个标题栏的例子来简单的体验一下创建自定义控件

### 1. 引入布局

在前端页面中有许多重复的页面，对于品字形的页面，导航栏、底部栏这些应该都是可以复用的页面，而在Android对于这种重复的页面也是可以复用的

在layout文件夹下新建一个title.xml

title.xml

```xml
<LinearLayout
    android:layout_height="wrap_content"
    android:layout_width="match_parent"
    android:background="@drawable/title_bg"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <Button
        android:id="@+id/title_back"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/back_bg"
        android:text="Back"
        android:textColor="#fff"/>

    <TextView
        android:id="@+id/title_text"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_weight="1"
        android:gravity="center"
        android:text="Title Text"
        android:textColor="#fff"
        android:textSize="24sp"/>

    <Button
        android:id="@+id/title_edit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/edit_bg"
        android:text="Edit"
        android:textColor="#fff"/>

</LinearLayout>
```

然后在activity_main.xml中使用include标签引入即可

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <include layout="@layout/title"/>

</LinearLayout>
```

这样就可以做到页面的复用

### 2. 创建自定义空控件

引入布局仅仅是做到了页面的复用，但是对于一些通用的响应事件，还是不能够完成的，例如对于标题栏来说back按钮都是返回操作，这就需要自定义控件来完成了

创建一个自定义类TitleLayout.java继承LinearLayout，并且重写里面的构造方法

```java
public class TitleLayout extends LinearLayout {

    public TitleLayout(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        LayoutInflater.from(context).inflate(R.layout.title, this);
        Button titleBack = findViewById(R.id.title_back);
        Button titleEdit = findViewById(R.id.title_edit);
        //响应事件
        titleBack.setOnClickListener((View view) -> {
            ((Activity)getContext()).finish();
        });
        titleEdit.setOnClickListener((View view) -> {
            Toast.makeText(getContext(), "Cliked Edit Button" ,
                    Toast.LENGTH_SHORT).show();
        });
    }
}
```

重写了LinearLayout中的带有两个参数的构造函数，在布局中引入TitleLayout控件就会调用这个构造函数，所以需要在构造函数内对标题栏布局进行动态加载，使用LayoutInflater来实现，from()方法可以构建出一个LinearLayout对象，然后调用inflate可以动态加载一个布局文件，里面传入两个参数

- 加载布局文件的id
- 给加载好的布局再添加一个父布局

然后在activity_main.xml中引入这个布局文件即可

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

<!--    <include layout="@layout/title"/>-->

    <com.example.uicustomviews.TitleLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

然后运行程序，和上面的效果一致，并且还绑定了响应事件