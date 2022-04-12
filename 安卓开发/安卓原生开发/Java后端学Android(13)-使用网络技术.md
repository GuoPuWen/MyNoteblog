# 使用WebView

WebView可以让在应用程序中展示一些网页，加载和显示网页都是浏览器的任务，但是需求又有明确指出，不允许打开系统的浏览器。所以WebView就是能帮助我们在页面中显示一个网页

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        WebView webView = findViewById(R.id.webView);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.setWebViewClient(new WebViewClient());
        webView.loadUrl("https://www.baidu.com");
    }
}
```

上面在活动中注册了WebView控件，要在Android中使用网络技术是需要在AndroidManifest.xml中注册的

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

# 使用HTTP访问网络

### 1 使用HttpURLConnection

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/send_btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send Request"/>

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <TextView
            android:id="@+id/response_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />


    </ScrollView>

</LinearLayout>
```

MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    private TextView tx;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tx = findViewById(R.id.response_text);

        Button btnSend = findViewById(R.id.send_btn);
        btnSend.setOnClickListener((View v) -> {
            new Thread(() -> {
                URL url = null;
                HttpURLConnection httpURLConnection = null;
                BufferedReader reader = null;
                try {
                    url = new URL("https://www.baidu.com");
                    httpURLConnection = (HttpURLConnection)url.openConnection();
                    httpURLConnection.setRequestMethod("GET");
                    httpURLConnection.setConnectTimeout(8000);
                    httpURLConnection.setReadTimeout(8000);
                    InputStream inputStream = httpURLConnection.getInputStream();
                    reader = new BufferedReader(new InputStreamReader(inputStream));
                    StringBuffer sb = new StringBuffer();
                    String line = "";
                    while ((line = reader.readLine()) != null){
                        sb.append(line);
                    }
                    Log.e("MainActivity", sb.toString());

                    showResponse(sb.toString());
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    if(reader != null){
                        try {
                            reader.close();
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }

                    if(httpURLConnection != null){
                        httpURLConnection.disconnect();
                    }
                }

            }).start();
        } );
    }

    private void showResponse(final String s) {
        runOnUiThread(() -> {
            tx.setText(s);
        });
    }
}
```

![image-20210517103854592](http://cdn.noteblogs.cn/image-20210517103854592.png)

### 2. 使用OkHttp

导入依赖

```xml
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
```



```	java
public class MainActivity extends AppCompatActivity {

    private TextView tx;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tx = findViewById(R.id.response_text);

        Button btnSend = findViewById(R.id.send_btn);
        btnSend.setOnClickListener((View v) -> {
            new Thread(() -> {
                OkHttpClient client = new OkHttpClient();
                Request request = new Request.Builder()
                        .url("https://www.baidu.com")
                        .build();
                Response response = null;
                try {
                    response = client.newCall(request).execute();
                    String responseBody = response.body().string();
                    showResponse(responseBody);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();
        } );
    }

    private void showResponse(final String s) {
        runOnUiThread(() -> {
            tx.setText(s);
        });
    }
}
```

