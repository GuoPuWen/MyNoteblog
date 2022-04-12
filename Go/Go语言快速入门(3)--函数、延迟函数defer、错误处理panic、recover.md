# 函数

- Go语言的函数语法为，当函数返回一个为命名的返回值或者没有返回值的时候，返回列表的圆括号可以省略

```go
func name(paramtre-list) (result-list){
    body
}
```

- 一个函数能够返回不止一个结果，例如下面一个非常简单的交换swap函数

```go
func swap(a int, b int) (int , int){
	return b, a
}

func main() {
	a := 1
	b := 2
	a ,b = swap(a, b)
	fmt.Println(a)
	fmt.Println(b)
}
```

- 函数变量也有类型，可以赋值给变量或者传递或者从其他函数中返回，函数变量可以像其他函数一样调用，函数类型的零值是nil空值

```go
func square(n int) int {
	return n * n
}
func negative(n int) int {
	return -n
}
func product(m, n int)int {
	return m * n
}

func main() {
	f := square
	fmt.Println(f(3))

	f = negative
	fmt.Println(f(3))
	fmt.Printf("%T\n", f)

	//f = product	//错误 不能把func(int, int) int赋值给 func(int) int
}
```

- 迭代变量在for循环中遇到的问题，这个问题类似于js中的闭包

```go
func main() {
	var slice []func() //定义一个函数变量类型的slice切片

	sli := []int{1, 2, 3, 4, 5}
	for _, v := range sli {
		fmt.Println(&v)
		slice = append(slice, func() {
			fmt.Println(v * v)
		})
	}
	for _, v := range slice {
		 v()
	}
}
//输出
0xc0000180a8
0xc0000180a8
0xc0000180a8
0xc0000180a8
0xc0000180a8
25
25
25
25
25

//解决
func main() {
	var slice []func() //定义一个函数变量类型的slice切片

	sli := []int{1, 2, 3, 4, 5}
	for _, v := range sli {
		fmt.Println(&v)
        //定义一个局部变量
		b := v
		slice = append(slice, func() {
			fmt.Println(b * b)
		})
	}
	for _, v := range slice {
		 v()
	}
}
```

已经可以从上面打印v的地址看出问题，在循环中创建的所有函数变量共享相同的变量它是一个可访问的存储位置，而不是固定的值，解决的办法很简单，只需要引入一个局部变量即可

# 延迟函数

Go中的延迟函数会在当前函数返回前执行传入的参数，它经常被用于关闭文件描述符、关闭数据库连接以及解锁资源，使用defer有以下几个注意点

- 调用顺序

```go
import "fmt"

func main() {
	defer fmt.Println("defer")
	fmt.Println("begin")
	fmt.Println("end")
}
//输出
begin
end
defer
```

- defer函数即使求值

```go
func g(i int){
	fmt.Println("g i:", i)
}

func f(){
	i := 100
	defer g(i)
	fmt.Println("begin i:", i)
	i = 200
	fmt.Println("end i:", i)
	return
}

func main() {
	f()
}
//输出结果
begin i: 100
end i: 200
g i: 100
```

g()函数虽然在f()函数返回时，但是传递给g()函数的参数还是100，也就是说defer函数会被延迟调用，但是传递给defer函数的参数会在defer语句处就准备好

- 反序调用

```go
func f() {
	defer fmt.Println("defer01")
	fmt.Println("begin")
	defer fmt.Println("defer02")
	fmt.Println("----")
	defer fmt.Println("defer03")
	fmt.Println("end")
	return
}

func main() {
	f()
}
//输出
begin
----
end
defer03
defer02
defer01
```

第一个defer函数最后被执行，可以猜测这是一种栈的结构

- 与return执行的先后顺序

defer、return与返回值三者的执行逻辑应该是：return最先执行，return负责将结果写入返回值中，接着defer进行一些收尾的工作，最后函数携带当前返回值退出

下面看两个例子：

```go
func test() int {	//放回值没有命名
	var i int
	defer func() {
		i++
		fmt.Println("defer1", i)
	}()
	defer func() {
		i++
		fmt.Println("defer2", i)
	}()
	return i
}

func main() {
	fmt.Println("return:", test())
}
//输出
defer2 1
defer1 2
return: 0
```

```go
func test() (i int) {	//有名返回值
	defer func() {
		i++
		fmt.Println("defer1", i)
	}()
	defer func() {
		i++
		fmt.Println("defer2", i)
	}()
	return i
}

func main() {
	fmt.Println("return:", test())
}
//输出
defer2 1
defer1 2
return: 2
```

因为return的操作不是一个原子操作，分为赋值和返回值的操作，对于例子1，因为返回值没有命名，所以return默认指定了一个返回值假设为s，首先将i赋值给s，后续的操作是针对i的自然不会影响s，对于例子2，因为是有名的返回值，所以每一次的defer的操作都是基于i的

# 错误处理机制

Go语言的错误处理机制，多返回值是前提条件，在Go中内置了一个error接口来处理错误

```go
// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
	Error() string
}

//下面是Go中内置的关于error接口的简单实现
func New(text string) error {        
    return &errorString{text}
}
// errorString is a trivial implementation of error.
翻译
// 把error转换成String是错误的简单实现
type errorString struct {        
    s string
}
func (e *errorString) Error() string { 
    return e.s
}

```

Go对错误的处理就是通过方法的返回值告诉开发者需要对错误进行判断和处理，错误是可见的

在go中内置了三个关键字：

- panic：运行时发生了异常
- defer：延迟函数，前面介绍过
- recover：可以中止panic造成的程序崩溃，是一个只能在defer中发挥作用的函数，在其他作用域中不会发挥作用

```go
func TestPanic() {
	panic("发生了异常，程序退出")
}
//输出
goroutine 1 [running]:
main.TestPanic(...)
        E:/GoProject/HelloProject/src/panicTest.go:6
main.main()
        E:/GoProject/HelloProject/src/panicTest.go:19 +0x45
```

```go
func TestDeferAndRecover() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Println("发生了异常，异常的信息为", err)
		}
	}()
	panic("发生了异常")
}
//输出
发生了异常，异常的信息为 发生了异常
```

下面看一个经典的实例，除法案例，除数不能为0

```go
func division(x, y int) (int , error){
	if y == 0 {
		return 0, errors.New("y is not zero")
	}
	z := x / y
	return z, nil
}

func main() {
	result, err := division(1, 0)
	if err != nil {
		fmt.Println("发生了异常，并捕获到了，异常的信息为", err)
		return
	}
	fmt.Println(result)
}
//输出
发生了异常，并捕获到了，异常的信息为 y is not zero
```

上面一个简单的例子说明了如何抛出异常，但是上面是可以预测的，如果面对无法预测的异常时，如何捕获呢？这就要用到recover了

```go
func division2(x, y int) (res int, err error){
	defer func(){
		if e := recover(); e != nil{
			err = e.(error)
		}
	}()
	res = x / y;
	return res, nil
}
func main() {
	result, err := division2(1, 0)
	if err != nil {
		fmt.Println("发生了异常，并捕获到了，异常的信息为", err)
		return
	}
	fmt.Println(result)
}
//输出
发生了异常，并捕获到了，异常的信息为 runtime error: integer divide by zero
```

当调用division2(1, 0)时，一定会报除0异常，通过在defer中使用recover函数来捕获发送的异常，如果不为空，将这个异常复制给返回结果的变量err

还有一点需要注意的是，在go中一旦某一个协程发生了panic而没有被recover，那么整个go程序都会终止，这和Java的多线程不一样，Java中某一个线程抛出了异常但是没有被捕获是不会影响主线程的