# 四种基本布局

### 1. 线性布局

线程布局是一种非常常用的布局，这个局部会将它所包含的控件在线性方向上一次排列。

线性布局使用LinearLayout来指定，并且使用android:orientation属性来指定是水平排列还是在垂直排列：

- `android:orientation：vertical`：垂直方向排列
- `android:orientation="horizontal"`：水平方向排列

下面通过一个实例来演示LinearLayout的使用

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button1"
        android:layout_gravity="top"
        android:text="button1"></Button>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button2"
        android:layout_gravity="center_vertical"
        android:text="button2"></Button>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:id="@+id/button3"
        android:text="button3"></Button>

</LinearLayout>
```

![image-20210502222524211](http://cdn.noteblogs.cn/image-20210502222524211.png)

在上面使用了android:layout_gravity属性指定对齐方式，当然需要注意各个控件的摆放方式

下面来说一说`android:layout_weight`属性。weight可以知道这是一个权重属性，看如下例子

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

    <EditText
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="3"
        android:hint="Type something"
        android:id="@+id/input_message"/>
    <Button
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"
        android:text="Send"
        android:id="@+id/send_button"/>

</LinearLayout>
```

![image-20210502223227765](http://cdn.noteblogs.cn/image-20210502223227765.png)

上述将android:layout_width都设置为0dp，但是因为指定了layout_weight属性，则android:layout_width不在生效，使用layout_weight能在不同的手机屏幕上进行适配，上面表示EditText占的比例为3/5，而Button所占的比例为2/5

一种更加常用的写法有

```xml
<EditText
          android:layout_width="0dp"
          android:layout_height="wrap_content"
          android:layout_weight="1"
          android:hint="Type something"
          android:id="@+id/input_message"/>
<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Send"
        android:id="@+id/send_button"/>
```

将后面的Button设置为wrap_content，则界面将会更加美观

### 2. 相对布局

RelativeLayout叫做相对布局，通过相对定位的方式出现在布局的任何位置。通过一个例子进行理解

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:text="button1"/>

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"
        android:text="button2"/>

    <Button
        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="button3"/>

    <Button
        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:text="button4"/>

    <Button
        android:id="@+id/button5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:text="button5"/>
</RelativeLayout>
```



![image-20210502224506888](http://cdn.noteblogs.cn/image-20210502224506888.png)

上面的button都是基于父元素进行定位的，相信应该非常好理解。

当然也可以基于控件进行定位，例如

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="button3"/>

    <Button
        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/button3"
        android:layout_toLeftOf="@id/button3"
        android:text="button1"/>

    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/button3"
        android:layout_toRightOf="@id/button3"
        android:text="button2"/>



    <Button
        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/button3"
        android:layout_toLeftOf="@id/button3"
        android:text="button4"/>

    <Button
        android:id="@+id/button5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@id/button3"
        android:layout_toRightOf="@id/button3"
        android:text="button5"/>
</RelativeLayout>
```

上述是基于button3进行定位的，也是非常好的理解，不过多描述

![image-20210502224949969](http://cdn.noteblogs.cn/image-20210502224949969.png)

### 3. 帧布局

FrameLayout又叫帧布局，这种布局没有方便的定位方式，所有的控件都默认的摆放在布局的左上角

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    tools:context=".MainActivity">
    <EditText
        android:id="@+id/text_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="This is TextView"/>

    <ImageView
        android:id="@+id/image_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@mipmap/ic_launcher"/>
</FrameLayout>
```

![image-20210502225720672](http://cdn.noteblogs.cn/image-20210502225720672.png)

可见所有的控件都被摆在了左上角

### 4. 百分比布局

前面说过可以使用layout_weight来实现按照比例指定控件的大小，这样的好处是可以在不同的分辨率的手机进行适配，但是可以发现只有线性布局支持这种功能。所以就引入了百分比布局，使得可以使用百分比的方式来指定控件的大小

百分比布局只为FrameLayout帧布局和RelativeLayout相对布局进行了扩展，提供了PercentFrameLayout和PercentRelativeLayout两个全新的布局。Android将百分比布局定义在了Support库中，为了版本的兼顾需要添加support库

在app/build.gradle文件中添加下面依赖，需要注意的是书上写的是compile关键字，这个是被划下划线的，所以被弃用了

```
implementation 'com.android.support:percent:24.2.1'
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.percent.PercentFrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/button1"
        android:text="Button1"
        android:layout_gravity="left|top"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />
    <!-- 这里之所以能使用app前缀就是因为刚刚定义了app的命名空间，和我们
       一直能使用android前缀的属性是一个道理-->
    <!-- 左上   -->

    <Button
        android:id="@+id/button2"
        android:text="Button2"
        android:layout_gravity="right|top"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />
    <!-- 右上   -->

    <Button
        android:id="@+id/button3"
        android:text="Button3"
        android:layout_gravity="left|bottom"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />
    <!-- 左下   -->

    <Button
        android:id="@+id/button4"
        android:text="Button4"
        android:layout_gravity="right|bottom"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"
        />

</android.support.percent.PercentFrameLayout>
```

上面使用了android.support.percent.PercentFrameLayout标签，发现直接跑出来一个异常

```
E/AndroidRuntime: Caused by: java.lang.ClassNotFoundException: Didn't find class "android.support.percent.PercentFrameLayout" on path: DexPathList
```

很明显，就是类没有找到，查看依赖中的包名

![image-20210503065328514](http://cdn.noteblogs.cn/image-20210503065328514.png)

所以很显然需要使用androidx.percentlayout.widget.PercentFrameLayout作为标签，这里应该是版本迭代的问题，书上讲的可能现在已经过时

![image-20210503065524568](http://cdn.noteblogs.cn/image-20210503065524568.png)

效果好像不是很明显，我截图的时候加了红边框