本系列为《Android 第一行代码第三版》读书笔记

# 一、Android的版本与组件

![image-20210428145645072](http://cdn.noteblogs.cn/image-20210428145645072.png)

由上面可见，Android的主要市场是在Android5.0版本以上。

Android的四大组件：

- Activity：是Android应用程序的门面，在应用中可见的东西都是放在Activity里的
- Service：在后台默默运行的组件
- BroadcastReceiver：允许应用接收各处的广播消息，比如电话、短信等
- ContentProvider：为应用程序之间共享数据提供帮助

# 二、Hello World

需要的基本条件：jdk、sdk、Android Studio

启动AS一路创建项目即可，选取创建一个空项目，注意选取语言为Kotlin，因为Google简易采用Kotlin来进行开发，同时指定Minimum SDK的版本为21，查找上面的版本表可以发现版本为21的为Android 5版本

创建一个空项目之后，AS已经默认生成了一个Hello World的Demo，只需要将程序进行打包运行即可，这里选择外部模拟器MuMu模拟器进行模拟运行。下载安装好MuMu模拟器，进入到如下目录

![image-20210428225930115](http://cdn.noteblogs.cn/image-20210428225930115.png)

使用命令行命令：

```
adb_server.exe connect 127.0.0.1:7555
```

那么这就将AS与MuMu模拟器连接起来了，在AS中点击运行键即可连接运行！

![MuMu20210428230206](http://cdn.noteblogs.cn/MuMu20210428230206.png)

那么程序运行成功！如上是我修改了Hello world为Hello，Android！之后的界面

# 三、项目目录分析

![image-20210428230452077](http://cdn.noteblogs.cn/image-20210428230452077.png)

熟悉过Java Web开发便可知，项目核心部分在app部分，也是需要我们编写代码的部分，因为此项目是采用gradle进行搭建的，所以有一些编译型的文件，这里不过多介绍，值介绍一些非常核心的目录

![image-20210428230647894](http://cdn.noteblogs.cn/image-20210428230647894.png)

- mipmap系：用来放置图标的，有多个版本的文件夹，是为了适应不同的设备，介绍为了兼容性
- values系：存放字符串、样式、颜色等配置的
- layout系：存放布局文件的
- drawable系：存放图片的
- AndroidManifest.xml：整个项目的配置文件，在程序中定义的组件都需要在这个文件中注册

# 四、项目运行流程

分析一个Android项目，首先是从AndroidManifest.xml开发的，因为这是整个项目的配置文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.hellowrold">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.HelloWrold">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

activity表示对MainActivity进行注册，而.表示省略包名，因为package中已经定义，其中intent-filter非常重要，看名字为一个拦截器，而其中定义了整个项目的主Activity，也就是项目的入口

接着分析MainActivity

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

虽然语言是kotlin，但是不影响阅读，首先MainActivity是继承AppCompatActivity的，AppCompatActivity是AndroidX中提供的一种向下兼容的Activity，使得Activity在不同版本中的功能保持一致。

Activity类时Android系统提供的一个基类，项目中所有定义的Activity都必须继承它或者它的子类才具有一个Activity的特性，然后onCreate是一个其中的方法，相当于生命周期的方法

接着方法里面调用了setContentView方法，Android讲究逻辑与视图分离，所以在Activity是不写界面的，界面是放在局部文件里面的。可以看到setContentView中引入了一个activity_main布局

那么切换到布局文件夹layout，果然有一个activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello ，Android！"	//这就是显示的字段
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

# 五、Android的日志工具Log

- Log.v() 。用于打印那些最为琐碎的、意义最小的日志信息。对应级别verbose，是Android日志里面级别最低的一种。
- Log.d() 。用于打印一些调试信息，这些信息对你调试程序和分析问题应该是有帮助的。对应级别debug，比verbose高一级。
- Log.i() 。用于打印一些比较重要的数据，这些数据应该是你非常想看到的、可以帮你分析用户行为数据。对应级别info，比debug高一级。
- Log.w() 。用于打印一些警告信息，提示程序在这个地方可能会有潜在的风险，最好去修复一下这些出现警告的地方。对应级别warn，比info高一级。
- Log.e() 。用于打印程序中的错误信息，比如程序进入到了catch语句当中。当有错误信息打印出来的时候，一般都代表你的程序出现严重问题了，必须尽快修复。对应级别error，比warn高一级。
  