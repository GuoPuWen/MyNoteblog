本系列开始为《罗剑锋的C++实战》阅读笔记。

# Code Style

看过很多计算机的书籍，但是在代码风格这块，很少有着文章有着介绍，自己的代码风格也完全是参照语言书籍上的代码示例形成的，但是对于程序来说，Code Style是必不可少的。

文章中提到一个观点，“好程序里的空白行至少要占到总行数的 20% 以上 ”，也就是说在变量=的前后，一段代码的前后需要通过空格或者空行来提高美感，例如下面，是一段代码没有任何的空格与空行

![image-20220312172410437](http://cdn.noteblogs.cn/image-20220312172410437.png)

加上一些空格或者空行之后

![image-20220312172423152](http://cdn.noteblogs.cn/image-20220312172423152.png)

“缓存失效与命名是计算机科学的两大难题 ”命名格式也是我经常头疼的一个事情，关于命名的风格，目前广泛使用的有三种

- 匈牙利命名法，在早期的 Windows 上很流行，使用前缀 i/n/sz 等来表示变量的类型，比如 iNum/szName。它把类型信息做了“硬编码”，不适合代码重构和泛型编程，所以目前基本上被淘汰了 。但是里面有一种做法我还是比较欣赏的，就是给成员变量加“m”前缀（member），给全局变量加“g_”前缀（global），比如 m_count、g_total，这样一看就知道了变量的作用域  
- 驼峰式命名法，就是单词首字母大写
- snake_case，用的是全小写，单词之间用下划线连接。这是 C 和 C++主要采用的命名方式，看一下标准库，里面的 vector、unordered_set、shrink_to_fit 都是这样  

文章中对变量命名提出了以下四条规则：

1. 变量、函数名和名字空间用 snake_case，全局变量加`g_`前缀；
2. 自定义类名用 CamelCase，成员函数用 snake_case，成员变量加`m_`前缀；
3. 宏和常量应当全大写，单词之间用下划线连接；
4. 尽量不要用下划线作为变量的前缀或者后缀（比如 local、name），很难识别  

# 预处理编程

对于C++程序，需要经过四个阶段：编码 -> 预处理 -> 编译 -> 运行。预处理阶段编程的操作目的是“源码”，用各种指令控制预处理器，把源码改造成另一种形式，就想是捏橡皮泥一样。

预处理都以符号“#”开头，虽然都在一个源文件里，但是不属于C++语言，走的是预处理器，不接收C++语法规则的约束

### #include

预处理指令“#include”，作用是包含文件，它可以包含任意的文件，#include是非常弱的，不做任何的代码检查，就是死脑筋的将数据合并进源文件，所以都是遵循Include Guard风格，来保护头文件，为了防止代码被重复包含

```c++
#ifndef _XXX_H_INCLUDE_
#define _XXX_H_INCLUDE_

#endif
```

同时对于一个大数组来说，里面有成百上千个数，放在文件里占用了很多地方，

```c++
static uint32_t calc_table[] = { // 非常大的一个数组，有几十行
    0x00000000, 0x77073096, 0xee0e612c, 0x990951ba,
    0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3,
    0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988,
    0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91,
    ...
};
```

可以使用include将其单独摘出来存放为一个`*.inc`的文件

```c++
// 比较大的数组
0x00000000, 0x77073096, 0xee0e612c, 0x990951ba,
0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3,
0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988,
0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91,

//使用
#include <iostream>

using namespace std;

static uint32_t calc_table[] = {
#include"calc_values.inc"
};
int main() {
   cout << calc_table[1];
   return 0;
}
```

### #define/#undef

`#define`在预处理阶段无所不能，可以无视C++语法限制，替换任何文字，定义常量/变量，实现函数功能，为类型起别名，减少重复代码等等，但是使用宏的时候一定要谨慎，时刻记着以简化代码、清晰易懂为目标，不要“滥用”，避免导致源码混乱不堪，降低可读性  

- 宏的展开、替换发生在预处理阶段，不涉及函数调用，参数传递、指针寻址，没有任何运行期的效率损失，所以对于一些调用频繁的小代码片段来说，用宏封装的效果比inline关键字要好

```c++
#define ngx_tolower(c) ((c >= 'A' && c <= 'Z') ? (c | 0x20) : c)	//变成小写
#define ngx_toupper(c) ((c >= 'a' && c <= 'z') ? (c & -0x20) : c)	//变成大写
```

- 宏没有作用域概念，永远都是全局生效，所以对于一些宏，最好是用完后尽快用`#undef`取消定义，避免冲突

```c++
#define CUBE(c) (c) * (c) * (c)
    std::cout << CUBE(10) << std::endl;
    std::cout << CUBE(15) << std::endl;

#undef CUBE
```

- 宏定义前先检查，如果有定义就先undef，然后在重新定义

```c++
#ifdef AUTH_PWD
#undef AUTH_PWD
#endif
#define AUTH_PWD 1000
```

- 定义常量

```c++
#define VERSION "1.0.18"
```

- 用宏来代替定义名字空间

```c++
#define BEGIN_NAMESPACE(x) namespace x {
#define END_NAMESPACE(x) }

BEGIN_NAMESPACE(my_own)
    
END_NAMESPACE(my_own)
```

### 条件编译 #if/#else/#endif

通过判断宏的数值来产生不同的源码，改变源文件的形态，这就是条件编译，通常编译环境都会有一些预定义宏，比如 CPU 支持的特殊指令集、操作系统 / 编译器 / 程序库的版本、语言特性等，使用它们就可以早于运行阶段，提前在预处理阶段做出各种优化，产生出最适合当前系统的源码  

`__cplusplus`，这个预定义宏标记了C++语言的版本号，使用它能够判断当前是c还是c++，是c++98还是c++11

```c++
#ifdef __cplusplus
    extern "C" {
#endif
    void a_c_function(int a);
#ifdef __cplusplus
    }
#endif

int main()
{
#if __cplusplus >= 201402
    cout << "c++14 or later" << endl;
#elif __cplusplus >= 201103
    cout << "c++11 or before" << endl;
#else   // __cplusplus < 201103
#   error "c++ is too old"
#endif  // __cplusplus >= 201402
   return 0;
}
```

除了`__cplusplus`，C++ 里还有很多其他预定义的宏，像源文件信息的`FILE` `LINE` ` DATE`，以及一些语言特性测试宏，比如`__cpp_decltype` `__cpp_decltype_auto`  `__cpp_lib_make_unique`等  

# 属性 attribute

`#define`，`#include`都是控制预处理器的命令，而属性则是用来控制编译器的编译指令。在C++11之前，标准里没有规定编译指令，在GCC里面实现了自己的编译指令，例如GCC里面的`__attribute__`，到了C++11，标准委员会认识到了编译指令的好处，于是起了个正式的名字叫属性。

属性没有新增关键字，而是使用两队方括号的形式`[[]]`，方括号的中间就是属性标签。C++11里面只定义了两个属性`noreturn`和`carries_dependency  `，基本上用处不大，到了C++14情况好了点，增加了一个比较使用的属性`deprecated`，用来标记不推荐使用的变量、函数或者类，也就是废弃，下面列出一些比较有用的属性

- deprecated：与 C++14 相同，但可以用在 C++11 里。
- unused：用于变量、类型、函数等，表示虽然暂时不用，但最好保留着，因为将来可能会用
- constructor：函数会在 main() 函数之前执行，效果有点像是全局对象的构造函数
- destructor：函数会在 main() 函数结束之后执行，有点像是全局对象的析构函数
- always_inline：要求编译器强制内联函数，作用比 inline 关键字更强
- hot：标记“热点”函数，要求编译器更积极地优化  

```c++
[[gnu::unused]]
int nouse;
```

# 静态断言 static_assert

使用assert用来断言一个表达式必定为真，例如

```c++
assert(i > 0 && "i must be greater than zero");
assert(p != nullptr);
assert(!str.empty());
```

当程序运行到assert语句是，就会计算表达式的值，如果是false，就会输出错误消息，然后调用abort()终止程序的执行，`assert`虽然是一个宏，但是在预处理阶段不生效，只是在运行阶段才生效，所以又叫动态断言

`static_assert`叫做静态断言，是一个专门的关键字，而不是宏，因为它只在编译时生效，运行阶段看不见，所以是静态的，编译器看到`static_assert`会去计算表达式的值，如果是false就会报错，导致编译失败，例如

```c++
static_assert(sizeof(long) >= 8, "must run on x64")
```

`static_assert`运行在编译阶段，只能看到编译时的常数和类型，看不到运行时的变量、指针、内存数据等，是“静态”的，所以不要简单地把 assert 的习惯搬过来用

例如下面的用法，因为变量只能在运行阶段出现，而在编译阶段不存在，所以静态断言无法处理

```c++
char* p = nullptr;
static_assert(p == nullptr, "some error.");
```

# 面向对象编程准则

- 尽量少用继承和虚函数，对于面向对象编程，它的关键点在于抽象和封装，而继承和多态并不是核心。如果完全没有继承关系，就可以让对象不必承受父辈的重担，也没有隐含的重用代码也会降低耦合度，让类更独立
- 控制继承的层次，如果继承深度超过三层，就说明“过渡设计”
- 积极使用final，final把它定义类，可以显式的警用继承，防止他人有意无意的产生派生类
- 使用default，C++类一共有六大函数，构造函数、析构函数、拷贝构造函数、拷贝复制函数、转移构造函数、转移赋值函数。虽然C++编译器会自动的为我们生成这些函数的默认实现，但是还是建议使用`default`的形式，明确告诉编译器应该实现这个函数

> 从行为上说，系统隐式定义的默认构造和用户亲自定义的一个空的默认构造函数，T () {}没有任何差别，但是当用户对T类型的对象进行值初始化时`T value()`，过程却完全不同，前者系统会首先对`value`的内存清理，然后在调用默认构造。而后者由于用户提供了默认构造，系统则会直接调用默认构造，由于过程的不同，导致初始化的结果可能不同

```c++
class DemoClass final {
public:
    DemoClass() = default; // 明确告诉编译器，使用默认实现
    ~DemoClass() = default; // 明确告诉编译器，使用默认实现
};
```

- 当想要禁止某个函数形式时，可以使用`=delete`形式，例如想要禁止对象拷贝

```c++
class DemoClass final {
public:
	DemoClass(const DemoClass&) = delete; // 禁止拷贝构造
	DemoClass& operator=(const DemoClass&) = delete; // 禁止拷贝赋值
};
```

- 委托构造。如果类有多个不同形式的构造函数，为了初始化成员肯定会有大量的重复代码，常见的做法是使用一个init()函数，将公共部分抽取出来，但是这种的效率和可读性较差，在C++11可以使用委托构造的新特性，一个构造函数直接调用另外一个构造函数

```c++
class Demo final {
private:
    int a;

public:
    Demo(int x) : a(x) {}

    Demo() : Demo(0) {}

    Demo(const string& s) : Demo(stoi(s)) {}
};
```

- 成员变量初始化，在 C++11 里，你可以在类里声明变量的同时给它赋值，实现初始化，这样不但简单清晰，也消除了隐患  

```c++
class DemoInit final
{
private:
   int          a = 0;
   string       s = "hello";
   vector<int>  v{1, 2, 3};
public:
    DemoInit() = default;
    ~DemoInit() = default;
};
```

- 类型别名，C++11扩展了关键字using的用法，增加了typedef的能力，可以定义类型别名。写类的时候，我们经常会用到很多外部类型，比如标准库里的 string、vector，还有其他的第三方库和自定义类型。这些名字通常都很长（特别是带上名字空间、模板参数），书写起来很不方便，这个时候我们就可以在类里面用 using 给它们起别名，不仅简化了名字，同时还能增强可读性  

```c++
class DemoClass final
{
public:
    using this_type = DemoClass; // 给自己也起个别名
    using kafka_conf_type = KafkaConfig; // 外部类起别名
public:
    using string_type = std::string; // 字符串类型别名
    using uint32_type = uint32_t; // 整数类型别名
    using set_type = std::set<int>; // 集合类型别名
    using vector_type = std::vector<std::string>;// 容器类型别名
private:
    string_type m_name = "tom"; // 使用类型别名声明变量
    uint32_type m_age = 23; // 使用类型别名声明变量
    set_type m_books; // 使用类型别名声明变量
private:
    kafka_conf_type m_conf; // 使用类型别名声明变量
};
```

