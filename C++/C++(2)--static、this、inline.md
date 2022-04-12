# static

- 当static作为函数中的静态变量，当变量声明为static时，空间将在程序的生命周期内分配，即使多次调用该函数，静态空间的地址也只分配一次，前一次调用中的遍历值通过下一次函数调用传递。也就是说该变量在内存中只有一份拷贝

```c++
#include<iostream>
using namespace std;
void demo(){
    static int count = 0;
    cout << count << " ";
    count++;
}
int main(void){
    for(int i = 0;i < 5;i++) demo();
    return 0;
}
输出
0 1 2 3 4
```

- 当static作为类中的静态变量，由于声明为static的变量只被初始化一次，因为它们在单独的静态存储中分配了空间，因此类中的静态变量**由对象共享。**对于不同的对象，不能有相同静态变量的多个副本。也是因为这个原因，静态变量不能使用构造函数初始化。

```c++
#include<iostream> 
using namespace std; 
class Apple { 
public: 
	static int i; 
	Apple(){ 
		// Do nothing 
	}; 
}; 

int Apple::i = 1; 

int main() { 
	Apple obj; 
	// prints value of i 
	cout << obj.i; 
    return 0;
}
```

- 当static作为类对象时，和变量一样，对象在声明为static时具有范围，直到程序的生命周期

```c++
#include<iostream> 
using namespace std; 

class Apple { 
	int i; 
	public: 
		Apple() { 
			i = 0; 
			cout << "Inside Constructor\n"; 
		} 
		~Apple() { 
			cout << "Inside Destructor\n"; 
		} 
}; 

int main() { 
	int x = 0; 
	if (x==0) { 
		Apple obj; 
	} 
	cout << "End of main\n"; 
    return 0;
}
//输出
Inside Constructor
Inside Destructor
End of main  
```

当对象声明为静态时

```c++
#include<iostream> 
using namespace std; 

class Apple { 
	int i; 
	public: 
		Apple() { 
			i = 0; 
			cout << "Inside Constructor\n"; 
		} 
		~Apple() { 
			cout << "Inside Destructor\n"; 
		} 
}; 

int main() { 
	int x = 0; 
	if (x==0) { 
		static Apple obj; 
	} 
	cout << "End of main\n"; 
    return 0;
}
//输出
Inside Constructor
End of main
Inside Destructor
```

- 当static作为类中的静态函数时，就像类中的静态数据成员或静态变量一样，静态成员函数也不依赖于类的对象。我们被允许使用对象和'.'来调用静态成员函数。但建议使用类名和范围解析运算符调用静态成员。允许静态成员函数仅访问静态数据成员或其他静态成员函数，它们无法访问类的非静态数据成员或成员函数。

```c++
#include<iostream> 
using namespace std; 

class Apple { 
    public: 
        int num;
        // static member function 
        static void printMsg() {
            cout<<"Welcome to Apple!";
        }
}; 
int main() { 
    
    Apple::printMsg(); 
} 
```

# this

this指针的用处：

- 一个对象的this指针并不是对象本身的一部分，不会影响sizeof(对象)的结果
- this作用域是在类内部，当在类的非静态成员函数中访问类的非静态成员的时候，编译器会自动将对象本身的地址作为一个隐含参数传递给函数。也就是说，即使你没有写上this指针，编译器在编译的时候也是加上this的，它作为非静态成员函数的隐含形参，对各成员的访问均通过this进行

一个例子：

```c++
#include <iostream>
#include <cstring>

using namespace std;
class Person {

public:
    typedef enum {
        BOY = 0, GIRL
    }SexType;
    Person(char *n, int a, SexType s){
        name = new char[strlen(n) + 1];
        strcpy(name, n);
        age = a;
        sex = s;
    }
    int get_age() const{
        return this->age;
    }
    Person& add_age(int a) {
        age += a;
        return *this;
    }

    ~Person(){
        delete [] name;
    }
private:
    char *name;
    int age;
    SexType sex;
};

int main()
{
   Person p("zhangsan",20,Person::BOY); 
   cout<<p.get_age()<<endl;
   cout<<p.add_age(10).get_age()<<endl;
   return 0;
}
```

this会被编译器解析成`A *const`还是`A const *`，其实很容易想到，this所指向的对象是可以修改的，但是this指针本身是不能被赋予其他值的，所以肯定是常量指针，即为`*const`

# inline

当程序执行函数调用指令时，CPU将存储该函数调用后的内存指令，将函数的参数复制到堆栈上，最后将控制权交给指定的函数，接着CPU执行代码，将函数返回值存储在预定义的内存位置或者寄存器上，并将控制权返回给调用函数。

这就有一个问题，如果函数的执行时间少于从调用者函数到被调用函数的切换时间，也就是说对于小功能函数，由于小功能的执行时间少于切换时间，因此会产生开销

而inline便是为了减少小功能函数的开销，调用内联函数，将在被调用时插入或者替换内联函数的整个代码，替换由C++编译器执行，如果内联函数很小，则可以提高效率，因为使用内联函数，省去了参数压栈、生成汇编语言的 CALL调用、返回参数、执行return等过程所花费的额外开销

**内联函数的优点：**

1. 不会发生函数调用开销。
2. 调用函数时，还节省了push / pop变量在栈上的开销。
3. 它还节省了从函数返回调用的开销。
4. 内联函数时，可以使编译器对函数主体执行特定于上下文的优化。对于正常的函数调用，这种优化是不可能的。通过考虑调用上下文和被调用上下文的流程可以获得其他优化。
5. 内联函数可能对于嵌入式系统有用（如果很小），因为内联函数所产生的代码少于函数调用的前导和返回。

**内联函数的缺点：**

1. 内联函数中添加的变量消耗了额外的寄存器，在内联函数之后，如果要使用寄存器的变量编号增加，则它们可能会增加寄存器变量资源利用的开销。这意味着当在函数调用点替换内联函数主体时，该函数使用的变量总数也会被插入。因此，将用于变量的寄存器数量也将增加。因此，如果函数内联后的变量数急剧增加，则肯定会导致寄存器利用率增加。
2. 如果使用太多的内联函数，则由于重复执行相同的代码，二进制可执行文件的大小将很大。
3. 过多的内联也会降低指令Cache命中率，从而降低了从高速缓存到主存储器的指令获取速度。
4. 如果有人更改了内联函数中的代码，则内联函数可能会增加编译时间开销，然后必须重新编译所有调用位置，这是因为编译器将需要再次替换所有代码以反映更改，否则它将继续使用旧功能。
5. 内联函数对于许多嵌入式系统可能没有用。因为在嵌入式系统中，代码大小比速度更重要。
6. 内联函数可能会导致崩溃，因为内联可能会增加二进制可执行文件的大小。内存溢出会导致计算机性能下降。