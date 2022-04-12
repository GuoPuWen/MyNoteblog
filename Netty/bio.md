### 一、BIO简介

同步并阻塞IO(传统IO模型)，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端需要启动一个线程进行处理，这样就会导致如果这个连接不做任何事情，只是在等待客户端的消息那么就会造成浪费

![image-20210115192810412](http://cdn.noteblogs.cn/image-20210115192810412.png)

### 二、BIO服务器端

```java
public class BIOServer {
    private static int port = 8089;

    public static void main(String[] args) throws IOException {
        ExecutorService executorService = Executors.newCachedThreadPool();  //多线程
        ServerSocket serverSocket = new ServerSocket(port);
        while(true){
            System.out.println("等待用户连接");
            Socket accept = serverSocket.accept();  //堵塞
            System.out.println("连接到一个客户端");
            executorService.submit(() -> {handle(accept);});
        }
    }

    private static void handle(Socket socket)  {
        InputStream inputStream = null;
        try {
            inputStream = socket.getInputStream();
            byte[] bytes = new byte[1024];
            while(true){
                System.out.println("read");
                int i = inputStream.read(bytes);    //堵塞
                if(i != -1){
                    System.out.println(new String(bytes));
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {

            try {
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

运行上面代码，使用telnet客户端工具连接 telnet 127.0.0.1 8089，然后按ctrl + ]可以进行字符串的输入

![image-20210121193333303](http://cdn.noteblogs.cn/image-20210121193333303.png)

因为上面代码使用了多线程，所以可以同时连接多个客户端，但是如果客户端不发数据那么该资源是被浪费掉了，所以BIO适合于连接数目小的应用，而后面的NIO则可以解决这个问题