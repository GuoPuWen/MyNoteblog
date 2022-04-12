

来源：java核心技术、博客

## 一、概念

##### 1. **反射机制定义**

Java 反射机制是在运行状态中，对于任意一个类，都能够获得这个类的所有属性和方法，对于任意一个对象都能够调用它的任意一个属性和方法。这种在运行时动态的获取信息以及动态调用对象的方法的功能称为 Java 的反射机制。

##### 2.**反射机制的作用**

1. 在运行时判断任意一个对象所属的类型
2. 在运行时构造任意一个类的对象
3. 在运行时判断任意一个类所具有的成员变量和方法
4. 在运行时调用任意一个对象的方法，甚至可以调用private方法。

##### 3.关于反射机制的类

1. java.lang.Class        代表一个类

2. java.lang.reflect.Constructor   代表类的构造方法
3. java.lang.reflect.Field       代表类的成员变量
4. java.lang.reflect.Method   代表类的方法
5. java.lang.reflect.Modifier    用于判断和获取某个类、变量或方法的修饰符

## 二、获取字节码

因为反射机制是在运行状态的时候获取，所以要先获得字节码(Class)对象，有3种方式。

##### 1. getClass() 

用这种方法必须要明确具体的类并且创建该类的对象。该方法是Object类下的

```java
Person person = new Person("zhangsan", "F", 23);
Class<? extends Person> personClass = person.getClass();
```

##### 2. .class 

如果 T 是任意的 Java 类型（或 void 关键字) T.class 将代表匹配的类对象。例如：

```java
Class<Person> personClass1 = Person.class;
System.out.println(personClass1);
Class<Integer> cls_int = int.class;
System.out.println(cls_int);
```

##### 3.Class.forName(classname)

这是Class类下的静态方法。当然， 这个方法只有在 className 是类名或接口名时才能够执行。否则，forName 方法将抛出一个 checkedexception ( 已检查异常）。无论何时使用这个方法， 都应该提供一个异常处理器。

```java
Class<?> cls = Class.forName("cn.review.Reflect.Person");
```

注意：Class 类实际上是一个泛型类。例如， Person.class 的类型是 Class<Person>。但是实际中可以省略泛型，忽略其类型参数。

## 三、Class类

##### 1.获取成员变量

-  Field[] getFields() ：获取所有public修饰的成员变量
- Field getField(String name)   获取指定名称的 public修饰的成员变量
- Field[] getDeclaredFields()  获取所有的成员变量，不考虑修饰符
- Field getDeclaredField(String name) 

##### 2.获取构造方法

- Constructor<?>[] getConstructors()  
- Constructor<T> getConstructor(类<?>... parameterTypes)  
- Constructor<T> getDeclaredConstructor(类<?>... parameterTypes)  
-  Constructor<?>[] getDeclaredConstructors()

##### 3.获取成员方法

-  Method[] getMethods()  
-  Method getMethod(String name, 类<?>... parameterTypes)  
- Method[] getDeclaredMethods()  
-  Method getDeclaredMethod(String name, 类<?>... parameterTypes)  

##### 4.获取全类名

-  String getName()  

##### 5.获取修饰符

- int getModifiers()

## 四、Field：成员变量

##### 1.设置值

 void set(Object obj, Object value)  

##### 2.获取值

Object  get(Object obj) 

##### 3.忽略访问权限修饰符的安全检查

setAccessible(true):暴力反射

## 五、Constructor:构造方法

##### 1. 创建对象

T newInstance(Object... initargs)  

## 六、Method：方法对象

##### 1.执行方法

Object invoke(Object obj, Object... args) 

##### 2.获取方法名称

String getName