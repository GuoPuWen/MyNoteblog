参考《Redis设计与实现》

# 一、什么是SDS

Redis是使用C语言进行编写，大家都知道C语言对于字符串有自己的字符类型char[]，但是Redis并没有采用C语言自带的字符类型，而是自己构建了动态字符串的抽象类型

在github上下载到Redis的源码，查看sds.h文件，发现了如下定义

```c
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

Redis为了满足不同长度的字符串可以使用不同大小的Header，从而节省内存，对于上面的sdshdr的结构来说吗，可以概括为下面一种抽象的数据结构

```java
struct sdshdr {
  // 记录buf数组中已使用字节的数量
  // 等于SDS所保存字符串的长度
  int len;
  
  // 记录buf数组中未使用字节的数量
  int free;
  
  // 字节数组，用于保存字符串
  char buf[];
};
```

也就是说sds在C原有类型的基础上封装了len、free这些属性。正是有这些属性的存在，使得Redis在字符串的存储上性能大幅提高

# 二、为什么是SDS？

### 2.1 常数复杂度获取字符串的长度

对于传统的C字符串来说，获取一个字符串的长度的时间复杂度为O(n)，例如对于str = '"Redis"'，C语言底层存储了一共6个字符，因为还有一个字符边界“\0”，那么如果调用strlen函数，计算长度则会经过如下过程

![image-20210421155845336](http://cdn.noteblogs.cn/image-20210421155845336.png)

所以对于C语言的strlen函数计算长度则时间复杂度为o(n)，但是SDS只需要常数级别的计算，因为在SDS里面已经封装好了len长度，只需要直接将这个长度进行返回便是字符串的长度。

设置和更新字符串长度的工作是在API在执行的过程中自动完成的

### 2.2 杜绝缓存区溢出

strcat函数可以将src字符串的内容拼接到dest字符串的末尾，函数原型为

```c
char *strcat(char *dest, const char *src);
```

C在执行这个函数的时候会假设用户已经为dest分配了足够dest+src的空间，如果假设不成立，那么会造成溢出，可能导致污染与dest连续的内存空间

而SDS则与其不同，因为维护了一个free变量，表示该SDS还剩余多少空间，当调用API进行strcat的时候，API首先会进行检查SDS的空间是否满足所需要的需求，如果不满足的话，API将会自动将SDS的空间扩展至执行修改的大小，那么就解决了缓存区溢出的问题

例如，现在SDS有一个字符串s = “Redis”，调用sdscat("Redis", " Cluster")，那么首选SDS会先扩展s的空间，然后再次执行拼接操作

![image-20210421163028873](http://cdn.noteblogs.cn/image-20210421163028873.png)

![image-20210421163038644](http://cdn.noteblogs.cn/image-20210421163038644.png)

需要注意的是，SDS不仅会对进行拼接操作，同时还会为SDS分配13字节得到未使用空间，这与SDS的空间分配策略有联系

### 2.3 减少修改字符串时带来的内存重分配次数

在C语言中，对于一个修改的操作，无论是增长还是缩短一个C字符串，程序总要对这个字符串进行重新的内存分配：

- 如果程序执行的是一个增长字符串的操作，那么需要通过内存重分配来扩展底层数组的大小，否则会产生缓冲区溢出
- 如果程序执行的是一个缩短字符串的操作，那么程序需要内存重分配来释放字符串不再使用的那一部分开年，否则会产生内存泄漏

那么在Redis中，经常会进行字符串的修改操作，而且可能是大量的频繁的，如果每次进行一次修改就要进行内存的重新分配的话，那么将会消耗很多时间，所以SDS事先了空间预分配和惰性空间释放的两种优化策略

- 空间预分配：在SDS的字符串需要进行字符串的增长操作的时候，如果SDS的长度小于1M，那么程序分配和len属性同样大小的空间，也就是如上看到的对s进行修改之后也同样分配13个字节的未使用空间；如果SDS的长度大于1M，程序会分配1M的未使用空间
- 惰性空间释放：惰性空间用于优化SDS的字符串的缩短操作，当SDS的API需要缩短保存的字符串的时候，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是free属性将这些字节的数量记录起来，并等待将来使用

### 2.4 二进制安全

C字符串中的字符必须符合某种编码，并且出来字符串的末尾之外，字符串里面不能包含空字符串，否则将会被认为是字符串结尾，这样就导致字符串只能存储文本数据，而不能保存像图片、视频这种二进制数据

而SDS就不存在这样的问题，虽然底层使用的还是C的char[]数组，但是它判断字符串的结尾是靠属性len来识别的，这样使得Redis不仅可以保存文本数据，还可以保存任意格式的二进制数据

### 2.5 兼容部分C字符串的函数

SDS一样遵循以空字符串结尾的惯例，所以可以使用c的一些字符串的函数库

### 2.6 总结

![image-20210421171233310](http://cdn.noteblogs.cn/image-20210421171233310.png)