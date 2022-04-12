参考：《Java编程思想》

IO属于java的基本知识，但是io的知识体系过于庞大，书中提到过“对程序语言的设计者来说，创建一个好的输入/输出系统是一项艰难的任务”，我觉得艰难在于，需要考虑太多的因素，因为文件有多种多样：文本格式，音视频格式，zip格式，网络等等，读取的方式也很多：顺序读取，随机读取，按行，按字符等等。这也就导致了Java io类库的庞大。

### 一、io的分类

Java的io在总体抽象上分为两类：

- 堵塞式io（bio）
- 非堵塞式io（nio）

学过操作系统，我们都知道一个系统的瓶颈大程度上取决于io的速度，io所花费的时间相比cpu处理的时间多太多了，堵塞式io在read时会一直等待读取端口的的内容，导致堵塞，write方法也同样如此，后面再网络编程这一块会大量的去比较这两种io的区别，在单机环境下，总体还是使用bio，也就是这篇文章要说到的输入输出流

根据应用场景分类

- 流式io

  流式io又分为

  - 字符流

  - 字节流

    字符流处理的单元为2个字节的Unicode字符，分别操作字符、字符数组或字符串，而字节流处理单元为1个字节，操作字节和字节数组

- 非流失io，辅助流式io

- 文件读取时与安全相关的类

总体上来说，io下有5个类/接口：磁盘操作：File，字节操作：InputStream 和 OutputStream，字符操作：Reader 和 Writer，对象操作：Serializable



### 二、输入字节流InputStream 

![](C:\Users\VSUS\Desktop\笔记\io\img\1.png)

（图来自网络）

- InputStream

  - ByteArrayInputStream
  - FileInputStream
  - FilterInputStream
    - BufferedInputStream
    - DataInputStream
    - LineNumberInputStream
    - PushbackInputStream

  - ObjectInputStream
  - PipedInputStream
  - StringBufferInputStream

①InputStream是所有输入流的父类，是一个抽象类

②ByteArrayInputStream、FileInputStream、StringBufferInputStream分别从字节数组，文件对象，String类中读取数据

③ObjectOutputStream和所有FilterOutputStream的子类都是装饰流。

> 装饰器模式：在不修改原先对象核心的功能的情况下，对功能进行增强。

![](C:\Users\VSUS\Desktop\笔记\io\img\2.png)

### 三、输出字节流OutputStream

![](C:\Users\VSUS\Desktop\笔记\io\img\3.png)

（图来自网络）

- OutputStream
  - ByteArrayOutputStream
  - FileOutputStream
  - FilterOutputStream
    - BufferedOutputStream
    - DataOutputStream
    - PrintStream
  - ObjectOutputStream
  - PipedOutputStream

①OutputStream是所有输入流的父类，是一个抽象类

②ByteArrayOutputStream、FileOutputStream、分别从字节数组，文件对象，中写入数据

③ObjectOutputStream和所有FilterOutputStream的子类都是装饰流。

![](C:\Users\VSUS\Desktop\笔记\io\img\4.png)



### 四、输入字符流Reader

![](C:\Users\VSUS\Desktop\笔记\io\img\6.png)

- Reader

  - CharArrayReader
  - BufferedReader
    - LineNumbeReader

  - FilterReader
    - PushbackReader

  - InputStreamReader
    - FileReader

  - PipeReader
  - StringReader

①Reader是所有类的基类

② InputStreamReader是一个连接字节流和字符流的桥梁，它将字节流转变为字符流。

### 五、输出字符流Writer

![](C:\Users\VSUS\Desktop\笔记\io\img\5.png)

（图来自网络）

- Writer
  - BufferedWriter
  - CharArrayWriter
  - FilterWriter
  - OutPutStreamWriter
    - FileWriter

  - PipedWriter
  - PrintWriter
  - StringWriter

①Writer是所有类的基类

②OutputStreamWriter是OutputStream到Writer转换的桥梁

### 六、高速缓冲字节流

当文件较大时，使用高速缓冲流能显著提高效率

- 写入数据到流中，字节缓冲输出流 BufferedOutputStream

- 读取流中的数据，字节缓冲输入流 BufferedInputStream

  通过构造方法就可以明白如何去使用缓冲流，只需将输入/输出字节流作为参数传入字节缓冲输入/输出流

```java
public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
    }

    /**
     * Creates a new buffered output stream to write data to the
     * specified underlying output stream with the specified buffer
     * size.
     *
     * @param   out    the underlying output stream.
     * @param   size   the buffer size.
     * @exception IllegalArgumentException if size &lt;= 0.
     */
    public BufferedOutputStream(OutputStream out, int size) {
        super(out);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
```

### 七、高速缓冲字符流

- 字符缓冲输入流 BufferedReader

- 字符缓冲输出流 BufferedWriter

使用方法也和缓冲字节流一致