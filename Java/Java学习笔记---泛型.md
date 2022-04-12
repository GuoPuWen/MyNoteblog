参考----《Java核心技术》
# 一、概述
在java增加泛型之前，通用程序的设计就是利用继承来实现的，将方法的类型参数设置为基类，这样的方法将会具有通用性，例如下面
	
```java
class ArrayList {
	private Object[] elementData;
	public Object get(int i) {
		
	}
	public void add(Object o) {
		
	}
}
```
这样的实现是有两个问题的：①当获取一个值时必须要进行强制类型转换；②这里可以添加任何Object对象，没有任何的错误检查。这里我们假定使用StringValue来存放String集合，ArrayList只是维护一个Object引用的数组，我们无法阻止将Integer类型加入到我们的String集合中当我们需要数据时，需要将获取的Object对象转换为我们期望的类型，如果向集合中添加了非预期的类型，编译时不会出现任何错误，但是在运行期则会报异常ClassCastException。

# 二、泛型的好处与本质
> **使用泛型的好处**
> ①类型安全
> ②消除强制类型转换
> **泛型的本质**
> ①把对象/集合里面的元素类型推迟到创建集合的时候
> ②类型参数化

# 三、泛型的格式
数据类型< 泛型的参数 > 对象 = new 数据类型 < 泛型的参数 >；
```java
ArrayList<String> list = new ArrayList<String>();
```
在jdk1.7之后出现了一个类型推断，即可以省略new后面的泛型类型
```java
ArrayList<String> list = new ArrayList<>();
ArrayList<String> list = new ArrayList();
```
对于类型推断，只看左边。例如
	
```java
ArrayList list = new ArrayList<String>();
list.add(1);
list.add("String");
```
这里不会出现错误，String在这里泛型参数化就没有作用了。

# 四、泛型类
**一个泛型类就是具有一个或者多个类型变量的类**
基本语法：class 类名称 < 泛型标识符 > 
```java
public class Pair<T> {
	T first;
	T second;
	Pair(T first , T second){
		this.first = first;
		this.second = second;
	}
	public T getFirst() {
		return first;
	}
	public void setFirst(T first) {
		this.first = first;
	}
	public T getSecond() {
		return second;
	}
	public void setSecond(T second) {
		this.second = second;
	}
	
}
```
# 五、泛型接口
泛型接口与泛型类使用基本相同
```java
interface GenericType<T> {
	T T; //Error
}
```
> 在接口中，所有的实例域都会被初始化为public static final
> 所有的方法都是public的
所以在上面实例中不可出现泛型变量，因为接口里都是不可变的变量。

# 六、泛型方法
```java
public static <T> T getMiddle(T[] t) {
		return t[t.length /2];
	}
```
使用时，将类型参数化
```java
	String[] str = {"A" , "B" ,"C"};
	String middle = Generic.<String>getMiddle(str);
```
# 七、泛型通配符？

## 1. ？无界通配符

1. 为什么要有？无界通配符

   例子：有一个父类 Animal 和几个子类，如狗、猫等，现在我需要一个动物的列表。初始想法是：

   ```java
   List<Animal> listAnimals
   ```

   但是正确的想法：

   ```java
   List<? extends Animal> listAnimals
   ```

通配符其实在声明局部变量时是没有什么意义的，但是当你为一个方法声明一个参数时，它是非常重要的。

```java
package cn.ArrayList;
import java.util.ArrayList;
import java.util.List;
/**
 * @author四五又十
 * @create 2020/2/13 20:17
 */
public class demo2 {

    static int countLegs (List<? extends Animal > animals ) {
        int retVal = 0;
        for ( Animal animal : animals ){
        }
        return retVal;
    }

    static int countLegs1 (List< Animal > animals ){
        int retVal = 0;
        for ( Animal animal : animals ){

        }
        return retVal;
    }

    public static void main(String[] args) {
        List<Dog> dogs = new ArrayList<>();
        // 不会报错
        countLegs( dogs );
        // 报错
        countLegs1(dogs);
    }
}
class Animal{
}
class Dog extends Animal{
}
```

**报错信息：Error:(32, 20) java: 不兼容的类型: java.util.List<cn.ArrayList.Dog>无法转换为java.util.List<cn.ArrayList.Animal>**

> ？和 T 的区别：
>
> ？和 T 都表示不确定的类型，区别在于我们可以对 T 进行操作，但是对 ？不行
>
> 也就是说,泛型通配符在方法中才是有意义的，T 是一个 确定的 类型，通常用于泛型类和泛型方法的定义，？是一个 不确定 的类型，通常用于泛型方法的调用代码和形参，不能用于定义类和泛型方法。

## 2.< ? extends E>

1. 概念：< ? extends E>是上界通配符 ，用 extends 关键字声明，表示参数化的类型可能是所指定的类型，或者是此类型的子类。

   这样做有两个好处：

   	1. 如果传入的类型不是 E 或者 E 的子类，编译不成功
    	2. 泛型中可以使用 E 的方法，要不然还得强转成 E 才能使用

## 3.< ? super E>

1. 概念：

   < ? super E>是下界通配符表示参数化的类型可能是所指定的类型，或者是此类型的父类型，直至 Object

# 八、泛型代码与虚拟机

**虚拟机上没有泛型类型对象 ——所有对象都属于普通类**

## 1.类型擦除
无论何时定义一个泛型类型，都自动的提供了一个相应的原始类型。原始类型的名字就是删除类型参数后的泛型类型名。擦除类型变量，并替换为限定类型（无限定类型使用Object）。例如上面的Pair泛型类的原始类型
```java
public class Pair {
	Object first;
	Object second;
	Pair(Object first , Object second){
		this.first = first;
		this.second = second;
	}
	public Object getFirst() {
		return first;
	}
	public void setFirst(Object first) {
		this.first = first;
	}
	public Object getSecond() {
		return second;
	}
	public void setSecond(Object second) {
		this.second = second;
	}
	
}
```
## 2.翻译泛型表达式、泛型方法
当程序调用泛型方法时，如果擦除返回类型，编译器插入强制类型转换。

```java
		Pair<Employee> buddies = ...
		Employee boddy = buddies.getFirst();
```
擦除getFirst()的返回类型后，编译器将自动插入Employee的强制类型转换。也就说，编译器将这个方法的调用翻译为2条虚拟机指令：
（1）对原始方法Pair.getFirst()的调用
（2）将返回的Object类型强制转换为Employee类型

> 总之，关于java泛型转换需要知道：
> ①虚拟机上没有泛型，只有普通类和方法
> ②所有的类型参数都只用他们的限定类型替换
> 桥方法被合成来保持多态
> 为保持类型安全性必要时插入强制类型转换








