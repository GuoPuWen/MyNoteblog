本系列为《Android 第一行代码第二版》读书笔记



当程序中有大量的数据需要显示时，因为屏幕有限能够一次性在屏幕上显示的并不多，所以需要借助ListView来进行展示，ListView允许用户通过手指上下滑动的方式将屏幕外的数据滚动到屏幕内，同时屏幕上的原有的数据将会滚出屏幕外

但是ListView已经不是目前的选择，而是使用RecyclerView这种控件做相同的效果，由于RecyclerView具有更强大的功能以及性能。但是ListView的学习能对RecyclerView学习有一定的帮助和理解

需要注意的是，本篇文章只是ListView和RecyclerView的入门教程文章，并没有涉及到底层原理的实现

# ListView

先做一个小例子，现在有如下数据需要在页面上显示

```java
{"Apple","Banana","Orange","Watermelon","Pear","Grape","Pineapple","Strawberry","Cherry", "Mango","Apple","Banana","Orange","Watermelon","Pear","Grape","Pineapple","Strawberry","Cherry","Mango"};
```

在activity_main.xml中使用ListVView

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private String [] data = {"Apple","Banana","Orange","Watermelon","Pear","Grape","Pineapple","Strawberry","Cherry",
            "Mango","Apple","Banana","Orange","Watermelon","Pear","Grape","Pineapple","Strawberry","Cherry",
            "Mango"};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ArrayAdapter<String> adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, data);
        ListView view = findViewById(R.id.list_view);
        view.setAdapter(adapter);
    }
}
```

数组中的数据是无法直接传递给ListView的，需要使用一个适配器，Android中原生提供了一些适配器的实现类例如ArrayAdapter，数组类型的适配器，因为数据都是字符串，所以泛型使用字符串

在适配器的构造函数中传入三个参数，其中simple_list_item_1只是一个android中已经定义好的布局文件

上面体验了一下ListView的简单用法，但是对于一些复杂的item(就是一个一个的子项)，需要重写适配器，而且对于一个子项目也要定义点击事件等等

首先定义一个list_item.xml，里面就很简单的一个TextView，这个布局文件也就是每一个item需要显示的，就是上面android.R.layout.simple_list_item_1

```xml
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="30sp"/>

</LinearLayout>
```

定义一个bean，这个bean类封装着上布局文件里的元素，由于就一个文本

```java
public class Bean {
    private  String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

接着实现一个适配器MyAdapter，因为需要通过适配器进行数据到ListView的绑定，所以需要向自己定义的适配器上传入数据和一个上下文对象，用于解析布局文件

MyAdapter.java

```java
public class MyAdapter extends BaseAdapter {

    //数据
    private List<Bean> data;
    //上下文对象
    private Context context;

    public MyAdapter(List<Bean> data, Context context) {
        this.data = data;
        this.context = context;
    }

    @Override
    public int getCount() {
        return data.size();
    }

    @Override
    public Object getItem(int position) {
        return data.get(position).getName();
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if(convertView == null){
            convertView = LayoutInflater.from(context).inflate(R.layout.list_item, parent, false);
        }
        TextView tx = convertView.findViewById(R.id.tv);
        tx.setText(data.get(position).getName());
        Log.e("listview", "getView" + position);
        return convertView;
    }
}
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private List<Bean> data = new ArrayList<>();

    private void initDate(){
        for (int i = 0; i < 100; i++) {
            Bean bean = new Bean();
            bean.setName("数据" + i);
            data.add(bean);
        }
    }


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initDate();
        ListView listView = findViewById(R.id.list_view);
        listView.setAdapter(new MyAdapter(data, this));
    }
}
```

运行程序效果如图

![image-20210505170654836](http://cdn.noteblogs.cn/image-20210505170654836.png)

并且查看日志可以发现，每次显示一条数据，就要调用Myadapter中的getView方法

![image-20210505170753864](Java后端学Android(7)-ListView和RecyclerView.assets/image-20210505170753864.png)

那就是每次显示一条数据就要调用getView方法中的convertView.findViewById方法，而这个是每条数据都一样的，所以可以将这个抽取出来，提高ListView的性能

可以定义一个ViewHolder类，这个类里面定义所有的组件，那么这里只有一个TextView，然后将这个ViewHoader存入convertView中，每次中convertView中拿便可

```java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    ViewHolder viewHolder;
    if(convertView == null){
        viewHolder = new ViewHolder();
        convertView = LayoutInflater.from(context).inflate(R.layout.list_item, parent, false);
        viewHolder.textView = convertView.findViewById(R.id.tv);
        convertView.setTag(viewHolder);
    }else{
        viewHolder = (ViewHolder)convertView.getTag();
    }

    viewHolder.textView.setText(data.get(position).getName());
    Log.e("listview", "getView" + position);
    return convertView;
}

private final class ViewHolder{
    TextView textView;
}
```



另外，还可以对每一个item定义一个点击事件，ListView里已经为我们提供了点击事件的方法，我们只需重写即可

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    initDate();
    ListView listView = findViewById(R.id.list_view);
    listView.setAdapter(new MyAdapter(data, this));

    listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            Log.e("listview", "onItemclick" + position);
        }
    });
}
```

**面试ListView的性能优化时你要回答的上来以下两点：①在ListView的Adapter中复用getView方法中的convertView ②使用静态内部类ViewHolder,用于对控件的实例存储进行缓存，减少findViewById的调用次数**



# RecyclerView

RecyclerView是一个增强版的ListView，前面已经说过就是ListView的一些性能问题和功能方面比如横向滚动等等是很难做到的，RecyclerView是Android官方推荐使用的，那么接下来就体验一下RecyclerView的使用

在使用之前，需要先导入相关的依赖，同样是为了兼容性问题

```
implementation 'androidx.recyclerview:recyclerview:1.1.0'
```



同样的套路，需要对每一个item进行一个布局

recyclerview_item.xml

```xml
<LinearLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android" >


    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="30sp"/>

</LinearLayout>
```

activity_main.xml

```xml
	<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

一个bean类

```java
public class Bean {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

适配器类

MyAdapter.java

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {

    private List<Bean> data = new ArrayList<>();
    private Context context;

    public MyAdapter(List<Bean> data, Context context) {
        this.data = data;
        this.context = context;
    }

    @NonNull
    @Override
    public MyAdapter.MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = View.inflate(context, R.layout.recyclerview_item, null);
        return new MyViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull MyAdapter.MyViewHolder holder, int position) {
        holder.tx.setText(data.get(position).getName());
    }

    @Override
    public int getItemCount() {
        return data == null ? 0 : data.size();
    }

    public class MyViewHolder extends RecyclerView.ViewHolder {

        private TextView tx;

        public MyViewHolder(@NonNull View itemView) {
            super(itemView);
            tx = itemView.findViewById(R.id.tv);
        }
    }
}
```

有了上面的Listview的使用经验，这里已经知道了需要一个MyViewHolder，里面用来封装用到的组件，而RecyclerView则将这个优化了，强制用户定义一个MyViewHolder，并且需要重写构造方法。同时MyAdapter还需要加入一个泛型，而这个泛型就是我们自定义实现的MyViewHolder

MyAdapter继承了RecyclerView.Adapter后有三个方法

- onCreateViewHolder：创建视图Holder
- onBindViewHolder：绑定数据，将每一个Holder绑定数据
- getItemCount：size大小

接着，在MainActivity中使用RecyclerView控件

```java
public class MainActivity extends AppCompatActivity {

    private List<Bean> data = new ArrayList<>();

    private void initData(){
        for (int i = 0; i < 1000; i++) {
            Bean bean = new Bean();
            bean.setName("数据" + i);
            data.add(bean);
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initData();
        RecyclerView rv = findViewById(R.id.rv);
        LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
        rv.setLayoutManager(linearLayoutManager);

        rv.setAdapter(new MyAdapter(data, this));
    }
}
```

唯一不一样的是，需要传入一个布局管理器，告诉RecyclerView是何种布局，这个时候就体现出RecyclerView的强大之处，一般有三种：

- GridLayoutManager：网格布局
- LinearLayoutManager：线性布局
- StaggeredGridLayoutManager：瀑布流布局

接着就是点击事件，RecyclerView是没有自带点击事件的，也就是说要自己实现点击事件，也好实现可以借助控件的点击事件方法，传入自己的逻辑

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {

    private List<Bean> data = new ArrayList<>();
    private Context context;

    public MyAdapter(List<Bean> data, Context context) {
        this.data = data;
        this.context = context;
    }

    @NonNull
    @Override
    public MyAdapter.MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = View.inflate(context, R.layout.recyclerview_item, null);
        return new MyViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull MyAdapter.MyViewHolder holder, int position) {
        holder.tx.setText(data.get(position).getName());
    }

    @Override
    public int getItemCount() {
        return data == null ? 0 : data.size();
    }

    public class MyViewHolder extends RecyclerView.ViewHolder {

        private TextView tx;

        public MyViewHolder(@NonNull View itemView) {
            super(itemView);
            tx = itemView.findViewById(R.id.tv);

            tx.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    if(onRecyclerItemListener != null){
                       
 //借助组件的点击事件，直接调用接口里面的方法
                        onRecyclerItemListener.onRecyclerItemListener(getAdapterPosition());
                    }
                }
            });
        }
    }

    private OnRecyclerItemListener onRecyclerItemListener;

    //将接口的实体类set进
    public void setOnRecyclerItemListener(OnRecyclerItemListener onRecyclerItemListener) {
        this.onRecyclerItemListener = onRecyclerItemListener;
    }

    //定义一个接口 用来外部实现点击逻辑
    interface OnRecyclerItemListener{
        void onRecyclerItemListener(int position);
    }
}
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private List<Bean> data = new ArrayList<>();

    private void initData(){
        for (int i = 0; i < 1000; i++) {
            Bean bean = new Bean();
            bean.setName("数据" + i);
            data.add(bean);
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initData();
        RecyclerView rv = findViewById(R.id.rv);
        GridLayoutManager linearLayoutManager = new GridLayoutManager(this, 3);
        rv.setLayoutManager(linearLayoutManager);

        MyAdapter myAdapter = new MyAdapter(data, this);
        //点击事件
        myAdapter.setOnRecyclerItemListener(new MyAdapter.OnRecyclerItemListener() {
            @Override
            public void onRecyclerItemListener(int position) {
                Log.e("RecyclerView", "点击事件" + position);
            }
        });
        rv.setAdapter(myAdapter);

    }
}
```

