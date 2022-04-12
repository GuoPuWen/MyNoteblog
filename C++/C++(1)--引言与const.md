# 引言

对于C++这门语言，接触过很多次，基本的使用都会(我是Java语言入门的)，但是考虑到以后的工作可能会使用到更多的C++和Go，所以还是决定再次系统的学习一下C++语言的一些语法，学习路线很简单，根据Github上的CPlusPlusThings项目学习搭配适量的写题。对于C/C++，估计很多的Java程序员都会对它产生敬佩也许是敬畏，包括我，但是对于我本人来说，在走过Java入门，学习过Spring的生态之后，其实我对于Java这门语言的兴趣稍微下降了，Java和C/C++同样是工具，但是Java更多做上层的东西，对于我而言喜欢底层较多一点。

![image-20220218094506922](http://cdn.noteblogs.cn/image-20220218094506922.png)

# 编译器与编译过程

编译器与IDE的区别，这里不做详述，对于C++程序来说，大名鼎鼎的编译器便是GCC。[GCC](https://www.gnu.org/software/gcc/)是一个编译器套件，包含C、C++、Objective-C、Fortran、Java、Ada、Go等等，而我们经常使用的g++/gcc便是其中一个组件

- GCC:GNU Compiler Collection(GUN 编译器集合)，它可以编译C、C++、JAV、Fortran、Pascal、Object-C等语言。
- gcc是GCC中的GUN C Compiler（C 编译器）
- g++是GCC中的GUN C++ Compiler（C++编译器）

其中gcc和g++的区别如下：

- 对于 *.c和*.cpp文件，gcc分别当做c和cpp文件编译（c和cpp的语法强度是不一样的）
- 对于 *.c和*.cpp文件，g++则统一当做cpp文件编译
- 使用g++编译文件时，**g++会自动链接标准库STL，而gcc不会自动链接STL**
- gcc在编译C文件时，可使用的预定义宏是比较少的
- gcc在编译cpp文件时/g++在编译c文件和cpp文件时（这时候gcc和g++调用的都是cpp文件的编译器），会加入一些额外的宏。
- 在用gcc编译c++文件时，为了能够使用STL，需要加参数 –lstdc++ ，但这并不代表 gcc –lstdc++ 和 g++等价，它们的区别不仅仅是这个。

对于一个简单的Hello World程序，只需要使用g++ helloworld.cpp便可执行编译，接着运行

```c++
#include<iostream>
using namespace std;
int main() {
    cout<<"hello world"<<endl;
    return 0;
}
```

但是背后的编译有着很丰富的过程，其中的每一步展开述说都需要很大的篇幅，这里简单记录一下

1. 预处理，生成.i的文件
2. 将预处理后的文件转换成汇编语言，生成.s文件
3. 汇编变为目标代码(机器代码)生成.o的文件
4. 连接目标代码,生成可执行程序

# const

常类型是指使用类型修饰符**const**说明的类型，常类型的变量或对象的值是不能被更新的

### 作用

- 可以定义常量，且必须进行初始化
- 类型检查
  - 这里经常与#define宏定义常量进行对比，#define宏定义常量没有类型，const可以进行类型检查
  - const定义的变量只有类型为整数或枚举，且以常量表达式初始化时才能作为常量表达式

> 常量表达式：值不会改变且在编译过程中就能够得到计算结果的表达式，能在编译时求值的表达式，例如：
>
> const int a1 = 10;          	 // a1是常量表达式
>
> const int a2 = a1 + 20;      // a2是常量表达式
>
> int a3 = 5;                  		// a3不是常量表达式
>
> const int a4 = a3;         	// a4不是常量表达式，因为a3程序的执行到达其所在的声明处时才初始化，所以变量a4的值程序运行时才知道。但编译没问题
>
> 其中一维数组的定义方式，类型说明符 数组名[常量表达式]

- 防止修改，起保护作用
- 可以节省空间，避免不必要的内存分配，const定义常量从汇编的角度来看，只是给出了对应的内存地址，而不是像`#define`一样给出的是立即数，const定义的常量在程序运行过程中只有一份拷贝，而`#define`定义的常量在内存中有若干个拷贝。

## const对象默认为文件局部变量

非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

```c++
// file1.cpp
int ext
// file2.cpp
#include<iostream>

extern int ext;
int main(){
    std::cout<<(ext+10)<<std::endl;
}
```

## 指针与const

指针常量与常量指针：

- 如果*const*位于`*`的左侧，则const就是用来修饰指针所指向的变量，即指针指向为常量；
- 如果const位于`*`的右侧，*const*就是修饰指针本身，即指针本身是常量

1)指针常量

- 对于指向常量的指针，不能通过指针来修改对象的值。
- 不能使用void`*`指针保存const对象的地址，必须使用const void`*`类型的指针保存const对象的地址。
- 允许把非const对象的地址赋值给const对象的指针，如果要修改指针所指向的对象值，必须通过其他方式修改，不能直接通过当前指针直接修改

2)常量指针

- const指针必须进行初始化，且const指针的值不能修改

```c++
#include<iostream>
using namespace std;
int main() {
    int val = 10;
    int num = 20;

    const int *ptr = &val;  //指针常量 可以不赋予初值
    *prt = 20;  //error
    int *ptr2 = &val;
    *ptr2 = 20; //ok

    int * const ptr3;    //error 常量指针 必须赋予初值
    int * const ptr4 = &val;    //error 常量指针 必须赋予初值
    *ptr4 = 30;     //ok 可以进行修改
    ptr4 = &num;     //error
}
```

### 参数为引用，为了增加效率同时防止修改

> 内部数据类型是编译器本来就认识的，不需要用户自己定义，如int,char,double
>
> 非内部数据类型不是编译器本来就认识的，需要用户自己定义才能让编译器识别，如enum，union，class、struct

对于非内部数据类型而言，void func(A a) 这样声明的函数注定效率比较低。因为函数体内将产生A 类型的临时对象用于复制参数a，而临时对象的构造、复制、析构过程都将消耗时间。

所以应该将void func(A a) ，修改为void func(A &a)，改为引用传递，因为“引用传递”仅借用一下参数的别名而已，不需要产生临时对象。但是“引用传递”有可能改变参数a，所以应该改为void func(const A &a)

但是对于内部数据而言，不要将“值传递”的方式改为“const 引用传递”。否则既达不到提高效率的目的，又降低了函数的可理解性。例如void func(int x) 不应该改为void func(const int &x)

