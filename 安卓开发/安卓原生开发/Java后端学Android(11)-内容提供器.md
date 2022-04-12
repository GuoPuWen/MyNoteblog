# 运行时权限

在Android6.0开始引入了运行时权限的功能，运行时权限是指用户在安装软件时不需要一次性授权所有的权限，而是在软件的使用过程中再对某一项权限进行申请

Android将权限分为两类：

- 普通权限：不会直接影响到用户的安全和隐私的权限，对于这部分权限，系统自动授权
- 危险权限：可能会涉及到用户的隐私或者对设备安全性造成影响的权限

下表中列出来的都是危险权限

![image-20210515094142998](http://cdn.noteblogs.cn/image-20210515094142998.png)

表中的每一个危险权限表示一个权限组，在进行权限处理的时候使用的是权限名，但是一旦用户同意授权了，那么该权限锁对应的权限组中的所有权限也会同时被授权

### 如何使用

先写一个没有使用运行时权限申请权限但是使用了危险权限的例子，这样程序会直接崩溃，因为CALL_PHONE是危险权限，必须用户手动同意

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/btn1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Make Call"/>



</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn1 = findViewById(R.id.btn1);
        btn1.setOnClickListener((View v) -> {
            Intent intent = new Intent(Intent.ACTION_CALL);
            intent.setData(Uri.parse("tel:10086"));
            startActivity(intent);
        } );
    }
}
```

运行程序，点击按钮，程序直接崩溃，因为需要申请运行时权限，在Android6.0之前的手机上，在AndroidManifest.xml中定义这么一个权限即可

```xml
<uses-permission android:name="android.permission.CALL_PHONE"/>
```

而在Android6.0开始这样做是不行的，否则直接抛出ava.lang.SecurityException: Permission Denial: 异常

下面是申请运行时权限的流程

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button btn1 = findViewById(R.id.btn1);
        btn1.setOnClickListener((View v) -> {
            if(ContextCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED){
                ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.CALL_PHONE}, 1);

            }else{
                call();
            }
        } );
    }

    public void call(){
        Intent intent = new Intent(Intent.ACTION_CALL);
        intent.setData(Uri.parse("tel:10086"));
        startActivity(intent);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode){
            case 1:
                if(grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                    call();
                }else{
                    Toast.makeText(this,"You denied the permission", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }
}
```

要申请权限首先第一步是要判断是否具有权限，如果没有权限则使用ActivityCompat.requestPermissions来申请权限，一共有三个参数：

- Activity的实例，使用this可以
- String数组，将要申请的权限名放入数组中即可
- 请求码：唯一值，这里传入1，后面会用到

然后用户界面弹出一个申请权限的框，那么很显然需要一个回调函数，当用户点击同意或者取消时应该怎么办，而onRequestPermissionsResult则是这个回调函数。判断用户是否同意权限，然后做相应的操作

# 访问其他程序中的数据

如果一个应用程序通过内容提供器对其数据提供了外部接口，那么其他的应用程序就可以对这部分数据进行访问，Android中的电电话簿、短信等都程序都提供了类似的访问接口

ContentResolver类是内容提供器的具体类，可以通过Context类中的getContentResolver()方法获取该类的实例，其中这个类提供了一些的CRUD的方法，详细介绍一下query方法的参数

```java
public final @Nullable Cursor query(@RequiresPermission.Read @NonNull Uri uri,
                                    @Nullable String[] projection, @Nullable String selection,
                                    @Nullable String[] selectionArgs, @Nullable String sortOrder) {
    return query(uri, projection, selection, selectionArgs, sortOrder, null);
}
```

- uri：指定查询某个应用程序下的某一张表，uri的格式为：content://com.example.app.provider/table1
- projection：指定查询的列名
- selection：指定where的约束条件
- selectionArgs：为where中的占位符提供具体的值
- sortOrder：指定查询结果的排序方式

下面是一个读取系统电话簿联系人的小案列

```java
public class MainActivity extends AppCompatActivity {

    private ArrayAdapter<String> adapter;

    private List<String> contactsList = new ArrayList<>();


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ListView listView = findViewById(R.id.contacts_view);
        adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, contactsList);
        listView.setAdapter(adapter);
        if(ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.READ_CONTACTS}, 1);

        }else{
            readContacts();
        }
    }
    public void readContacts(){
        Cursor query = getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null);
        if (query != null){
            while (query.moveToNext()){
                String displayName = query.getString(query.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                String number = query.getString(query.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
                contactsList.add(displayName + "\n" + number);
            }
        }
        adapter.notifyDataSetChanged();
        query.close();
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode){
            case 1:
                if(grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                    readContacts();
                }else{
                    Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show();
                }
        }
    }
}
```

![image-20210515152154687](http://cdn.noteblogs.cn/image-20210515152154687.png)

# 创建自己的内容提供器

首选需要准备两个项目，内容提供者项目就在之前DatabaseTest的基础上进行，内容使用者新建一个项目

新建一个类DatabaseProvider需要继承ContentProvider这个类，这个抽象类具备6个方法需要我们重写，onCreate()、query()、insert()、update()、delete()、getType()，其中的每一个方法都会带上一个uri参数，而这个参数就是上面在使用的时候传入的uri参数，一个标准的uri是：

content://com.example.app.provider/table1

这里表示期望访问的是com.example.app这个app的table1数据，还可以加上一个id例如

content://com.example.app.provider/table1/1表示期望访问的是table1表中的id为1的数据。

UriMatcher中提供了一个addURI()的方法，接收三个参数authority、path、自定义代码其实很好理解，就是靠这个自定义的代码来确定具体访问的资源

getType()方法之前没有接触过，这是所有的内容提供者都必须提供的方法，用于获取Uri对象的MEMI类型，一个内容URI对应的MIMI字符串主要由三部分组成：

- 必须要以vnd开头
- 如果URI以路径结尾则后接 android.cursor.dir/，如果URI以id结尾，则后接android.cursor.item
- 最后接上`vnd.<authority>.<path>`

直接看代码即可

```java
public class DatabaseProvider extends ContentProvider {

    public static final int BOOK_DIR = 0;

    public static final int BOOK_ITEM = 1;

    public static final int CATEGORY_DIR = 2;

    public static final int CATEGORY_ITEM = 3;

    public static final String AUTHORITY = "com.example.databasetest.provider";

    private static UriMatcher uriMatcher;

    private MyDatabaseHelper dbHelper;

    static {
        uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
        uriMatcher.addURI(AUTHORITY, "Book", BOOK_DIR);
        uriMatcher.addURI(AUTHORITY, "Book/#", BOOK_ITEM);
        uriMatcher.addURI(AUTHORITY, "Category", CATEGORY_DIR);
        uriMatcher.addURI(AUTHORITY, "Category/#", CATEGORY_ITEM);
    }



    public DatabaseProvider() {

    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        int deleteRows = 0;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                deleteRows = db.delete("Book", selection, selectionArgs);
                break;
            case BOOK_ITEM:
                String bookId = uri.getPathSegments().get(1);
                deleteRows = db.delete("Book", selection, selectionArgs);
                break;
            case CATEGORY_DIR:
                deleteRows = db.delete("Category", selection, selectionArgs);
                break;
            case CATEGORY_ITEM:
                String categoryId = uri.getPathSegments().get(1);
                deleteRows = db.delete("Category","id + ?", new String[]{categoryId});
                break;
        }
        return deleteRows;
    }

    @Override
    public String getType(Uri uri) {
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                return "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book";
            case BOOK_ITEM:
                return "vnd.android.cursor.item/vnd.com.example.databasetest.provider.book";
            case CATEGORY_DIR:
                return "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.category";
            case CATEGORY_ITEM:
                return "vnd.android.cursor.item/vnd.com.example.databasetest.provider.category";
        }
        return null;

    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        Uri uriReturn = null;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
            case BOOK_ITEM:
                long bookId = db.insert("Book", null, values);
                uriReturn = Uri.parse("content://" + AUTHORITY + "/book/" + bookId);
                break;
            case CATEGORY_DIR:
            case CATEGORY_ITEM:
                long categoryId = db.insert("Category", null, values);
                uriReturn = Uri.parse("content://" + AUTHORITY + "/Category/" + categoryId);
                break;
            default:
        }
        return uriReturn;
    }

    @Override
    public boolean onCreate() {
        dbHelper = new MyDatabaseHelper(getContext(), "BookStore.db", null, 2);
        return true;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        SQLiteDatabase db = dbHelper.getReadableDatabase();
        Cursor cursor = null;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                cursor = db.query("Book", projection, selection, selectionArgs, null, null, sortOrder);
                break;
            case BOOK_ITEM:
                //获取表中数据的id
                String bookId = uri.getPathSegments().get(1);
                cursor = db.query("Book", projection, "id = ?", new String[]{bookId}, null, null, sortOrder);
                break;
            case CATEGORY_DIR:
                cursor = db.query("Category", projection, selection, selectionArgs, null, null, sortOrder);
                break;
            case CATEGORY_ITEM:
                String categoryId = uri.getPathSegments().get(1);
                cursor = db.query("Category", projection, "id = ?", new String[]{categoryId}, null, null, sortOrder);
                break;
            default:
                break;
        }
        return cursor;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        SQLiteDatabase db = dbHelper.getWritableDatabase();
        int updateRows = 0;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                updateRows = db.update("Book", values, selection, selectionArgs);
                break;
            case BOOK_ITEM:
                String bookId = uri.getPathSegments().get(1);
                updateRows = db.update("Book", values, selection, selectionArgs);
            case CATEGORY_DIR:
                updateRows = db.update("Category", values, selection, selectionArgs);
                break;
            case CATEGORY_ITEM:
                String categoryId = uri.getPathSegments().get(1);
                updateRows = db.update("Category", values, "id = ?", new String[]{categoryId});
        }
        return updateRows;
    }
}
```

当然不要忘记了内容提供者需要在AndroidManifest.xml中注册

```xml
<provider
          android:name=".DatabaseProvider"
          android:authorities="com.example.databasetest.provider"
          android:enabled="true"
          android:exported="true">
</provider>
```

接着新建一个ProviderTest项目，用来调用DatabaseTest程序

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
        android:id="@+id/add_data"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ADD TO BOOK"/>

    <Button
        android:id="@+id/query_data"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Query From BOOK"/>

    <Button
        android:id="@+id/update_data"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Update Book"/>

    <Button
        android:id="@+id/delete_data"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Delete From Book"/>


</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    public static final String AUTHORITY = "content://com.example.databasetest.provider/";

    private String newId = "";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button addBtn = findViewById(R.id.add_data);
        addBtn.setOnClickListener((View v) -> {
            Uri uri = Uri.parse(AUTHORITY + "Book");
            ContentValues contentValues = new ContentValues();
            contentValues.put("name", "A Clash of Kings");
            contentValues.put("author", "George Martin");
            contentValues.put("pages", 1040);
            contentValues.put("price", 22.85);
            Uri insert = getContentResolver().insert(uri, contentValues);
            newId = insert.getPathSegments().get(1);
        });

        Button queryBtn = findViewById(R.id.query_data);
        queryBtn.setOnClickListener((View v) -> {
            Uri uri = Uri.parse(AUTHORITY + "Book");
            Cursor query = getContentResolver().query(uri, null, null, null, null);
            if(query != null){
                while (query.moveToNext()){
                    String name = query.getString(query.getColumnIndex("name"));
                    String author = query.getString(query.getColumnIndex("author"));
                    int pages = query.getInt(query.getColumnIndex("pages"));
                    double price = query.getDouble(query.getColumnIndex("price"));
                    Log.e(TAG, "Book name is " + name);
                    Log.e(TAG, "Book author is " + author);
                    Log.e(TAG, "Book page is " + pages);
                    Log.e(TAG, "Book pages is " + price);
                }
            }
            query.close();
        });

        Button updateBtn = findViewById(R.id.update_data);
        updateBtn.setOnClickListener((View v) -> {
            Uri uri = Uri.parse(AUTHORITY + "Book/" + newId);
            ContentValues contentValues = new ContentValues();
            contentValues.put("name", "A Storm of Swords");
            contentValues.put("pages", 1216);
            contentValues.put("price", 24.05);
            getContentResolver().update(uri, contentValues, null, null);
        } );

        Button deleteBtn = findViewById(R.id.delete_data);
        deleteBtn.setOnClickListener((View v) -> {
            Uri uri = Uri.parse(AUTHORITY + "Book/" + newId);
            getContentResolver().delete(uri, null, null);
        } );
    }
}
```

最后结果查看日志即可