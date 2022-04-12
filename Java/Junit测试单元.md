## 1.概述

在平时写代码的时候，很多时候为了验证一些方法的结果是不是我们所需要的，会使用System.out方法来验证自己的代码是否正确。例如，写一个加法

```java
public int add(int a,int b){
    return a+b;
}
```

如果我们需要严重该方法的正确性，那么需要在main方法里new这个方法的类去调用这个方法，然后通过输出语句来判断代码是否正确，这个过程是比较繁琐的。

Junit单元测试框架就很好的解决问题。Junit方法在内部还提供了一个断言机制。可以来判断结果是否满足我们的期望。

## 2.使用方法

在导入Junit框架后，直接在方法上添加@Test方法

![](junit\1.png)

## 3.导入框架

1. 直接在需要测试的方法上@Test，点小灯泡，直接在idea中导入，不过这种自动导入的方法，我一直没有成功过

2. 手动导入，在网上下载Junit框架jar包，

   Project Settings  ---> 

![](junit\2.png)

选择JARS or directories，导入下载jar包即可

## 4.一些规范

1. 定义一个测试类(测试用例)
			* 建议：
				
		测试类名：被测试的类名Test		CalculatorTest

		包名：xxx.xxx.xx.test		cn.itcast.test

2. 定义测试方法：可以独立运行
   * 建议：
     * 方法名：test测试的方法名		testAdd()  
     * 返回值：void
     * 参数列表：空参

## 5.断言

使用断言方法来判断期望值与结果值是否一样，

```java
@Test
public void test_demo1(){
    Calculator calculator = new Calculator();
    int add = calculator.add(1, 2);
    Assert.assertEquals(3, add);
}
```