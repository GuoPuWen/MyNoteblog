Android中主要提供了三种方式用于简单的实现数据持久化的方式：文件存储、SharedPreference、以及数据库存储，下面一一介绍

# 文件存储

### 1. 数据存储到文件中

Context类中提供了一个openFileOutput方法用于将数据存储到指定的文件中

```java
@Override
public FileOutputStream openFileOutput(String name, int mode)
    throws FileNotFoundException {
    return mBase.openFileOutput(name, mode);
}
```

- name：文件创建的时候的文件名
- mode：模式：主要有两种模式可以选择，MODE_PRIVATE：默认的操作模式，当指定的文件存在时，会直接覆盖里面的内容，MODE_APPEND：追加，当指定的文件存在时，会往文件中追加内容

有了这个方法，一切就很简单了，使用Java流操作便可将数据写入文件中

```java
public class MainActivity extends AppCompatActivity {

    private EditText editText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        editText = findViewById(R.id.edit);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        String s = editText.getText().toString();
        save(s);
    }

    private void save(String s)  {
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try {
            out = openFileOutput("data", Context.);
            writer = new BufferedWriter(new OutputStreamWriter(out));
            writer.write(s);
        }catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(writer != null){
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

上面案例只有一个输入文本框，当关闭程序的时候调用onDestroy方法，将数据保存在data文件中，打开AS的Device File Explore窗口在view工具栏里面

![image-20210510225247343](http://cdn.noteblogs.cn/image-20210510225247343.png)

### 2. 将数据读出

Context类中提供了一个openFileInput方法，用于从文件中读取数据，用法也是直接使用Java流，下面将在应用启动的时候加载上一次的数据

```java
public class MainActivity extends AppCompatActivity {

    private EditText editText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String load = load();
        editText = findViewById(R.id.edit);
        if(!TextUtils.isEmpty(load)){
            editText.setText(load);

        }


    }

    public String load(){
        FileInputStream in = null;
        BufferedReader reader = null;
        StringBuilder builder = null;
        try {
             in = openFileInput("data");
             reader = new BufferedReader(new InputStreamReader(in));
             builder = new StringBuilder();
             String line = "";
             while((line = reader.readLine()) != null){
                 builder.append(line);
             }

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                reader.close();
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return builder.toString();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        String s = editText.getText().toString();
        save(s);
    }

    private void save(String s)  {
        FileOutputStream out = null;
        BufferedWriter writer = null;
        try {
            out = openFileOutput("data", Context.MODE_PRIVATE);
            writer = new BufferedWriter(new OutputStreamWriter(out));
            writer.write(s);
        }catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(writer != null){
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

# SharedPreferences存储

### 写入数据

SharedPreferences是靠键值对来进行数据存储的，当保存一条数据的时候，需要给这条数据提供一个键值对，然后在读取数据的时候就可以通过这个键将相应的值取出来，并且SharedPreferences提供了很多不同的数据类型

使用SharedPreferences存储对象需要先获取SharedPreferences对象，有三种方法获取SharedPreferences对象：

- Context类中的getSharedPreferences方法：
  - name：指定的文件名
  - mode：目前只支持一种，MODE_PRIVATE

```java
@Override
public SharedPreferences getSharedPreferences(String name, int mode) {
    return mBase.getSharedPreferences(name, mode);
}
```

- Activity类中的getPreferences()方法，不过这个只接受一个参数，默认以当前活动的类名作为SharedPreferences的文件名
- PreferenceManager类中的getDefaultSharedPreferences方法

例子：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn = findViewById(R.id.btn1);
        btn.setOnClickListener((View v) -> {
            SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
            editor.putString("name", "tom");
            editor.putInt("age", 28);
            editor.apply();
        });
    }
}
```

同样的打开Android Device Monitor窗口查看数据的保存位置SharedPreferences一般保存在/data/data/<包名>/shared_prefs目录下

![image-20210511222421305](http://cdn.noteblogs.cn/image-20210511222421305.png)

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="name">tom</string>
    <int name="age" value="28" />
</map>
```

可以很明显的看出来，是靠xml文件进行保存的，并且其中的数据格式也是靠各个标签进行管理的

### 读取数据

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn = findViewById(R.id.btn1);
        //写入数据
        btn.setOnClickListener((View v) -> {
            SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
            editor.putString("name", "tom");
            editor.putInt("age", 28);
            editor.apply();
        });
        //读取数据
        Button btn2 = findViewById(R.id.btn2);
        btn2.setOnClickListener((View v) -> {
            SharedPreferences sharedPreferences = getSharedPreferences("data", MODE_PRIVATE);
            String name = sharedPreferences.getString("name","");
            int age = sharedPreferences.getInt("age", 0);
            Log.e("MainActivity", name);
            Log.e("MainActivity", String.valueOf(age));
        });
    }
}
```

# SQLite数据库存储

Android内置了一个数据库，SQLite是一款轻量级的关系型数据库，运算速度非常快，占用资源少，支持标准的SQL语法，并且遵循数据库的ACID事务

