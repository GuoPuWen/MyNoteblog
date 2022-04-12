# 数组

数组的简单使用

```go
	var a [3]int = [3]int{1, 2, 3}
	var b [2]int = [2]int{1, 2}
	fmt.Println(a[0])

	//输出数组的值
	for i, j := range a {
		fmt.Printf("%d %d\n", i , j);
	}

	//仅输出元素
	for _, v := range a {
		fmt.Printf("%d", v)
	}
```

对于数组有以下注意的点：

- 数组长度是数组类型的一部分，不同长度的类型不一样

```go
package main

import "fmt"

func main() {
	var a [3]int = [3]int{1, 2, 3}
	var b [2]int = [2]int{1, 2}

	//输出类型
	fmt.Printf("type of a: %T\n",a)
	fmt.Printf("type of b: %T\n",b)

}
//输出
type of a: [3]int
type of b: [2]int
```

- 如果一个元素类型是可以比较的，那么这个数组也是可以比较的，也就是说可以直接使用“==''进行比较

```go
var a [3]int = [3]int{1, 2, 3}
var b [3]int = [3]int{1, 2, 3}

fmt.Println(a == b)
//输出
true
```

- 对数组的修改是值传递的，如果要进行对数组的修改，可以使用指针

```go
package main

import "fmt"

func set(nums [3]int){
	nums[2] = 100
}
//值传递
func print(nums [3]int){
	for _, v := range nums {
		fmt.Printf("%d\t", v)
	}
	fmt.Println()
}

func main() {
	var a [3]int = [3]int{1, 2, 3}
	print(a)
	set(a)
	print(a)
}
```

引用传递的例子

```go
func set(nums *[3]int){
	nums[2] = 100
}

func print(nums [3]int){
	for _, v := range nums {
		fmt.Printf("%d\t", v)
	}
	fmt.Println()
}

func main() {
	var a [3]int = [3]int{1, 2, 3}
	print(a)
	set(&a)
	print(a)
}
```

# slice

数组在Go语言中并没有那么常用，更常用的数据结构是切片，也就是动态数组slice，使用slice可以向slice中追加元素，在容量不足的时候会进行扩容

slice是一种轻量型的数据结构，可以用来访问数组的部分或者全部元素，slice有三个属性：指针、长度、容量。

`指针`指向数组中的第一个可以从slice中访问的元素，这个元素并不一定是数组的第一个元素

`长度`是指slice中的元素个数，它不能超过slice容量

`容量`通常是从slice的起始元素到底层数组的最后一个元素间的个数

slice包含了指向数组元素的指针，也就是说将一个slice传递给一个函数的时候，是引用传递的，函数内部可以修改底层数组的元素，例如reverse反转函数

```go
//反转
func reverse(s []int) {
	for i, j := 0, len(s) - 1; i < j; i, j = i + 1, j - 1 {
		s[i], s[j] = s[j], s[i]
	}
}
func main() {
	a := []int{1, 2, 3, 4, 5}
	reverse(a)
	fmt.Println(a)
}
//输出
[5 4 3 2 1]
```

将一个左移n个元素的做法是连续调用reverse函数三次，第一次反转前n个元素，第二次反转剩下的元素，最后对一整个slice进行反转

```go
//反转
func reverse(s []int) {
	for i, j := 0, len(s) - 1; i < j; i, j = i + 1, j - 1 {
		s[i], s[j] = s[j], s[i]
	}
}
func main() {
	a := []int{1, 2, 3, 4, 5}
	//向左移动两个元素 [3, 4, 5, 2, 1]
	reverse(a[:2])
	reverse(a[2:])
	reverse(a)
	//reverse(a)
	fmt.Println(a)
}
```

数组的类型如果相同是可以进行比较的，但是slice是不能比较的，所以不能使用==测试两个slice是否具有相同的元素，主要有两个原因：

- slice的元素是非直接的，有可能slice可以包含本身
- slice的元素不是直接的，所以如果底层数组元素发生改变，同一个slice在不同的时间内会具有不同的元素

slice可以和nil进行比较，值为nil的slice没有对应的底层数组，值为nil的slice长度和容量都为0，但是也有非nil的slice长度和容量是0

```go
var s []int //len(s) == 0 s == nil
s = nil 	//len(s) == 0 s == nil
s = []int(nil) 	//len(s) == 0, s == nil
s = []int{}		//len(s) == 0, s != nil
```

所以，检查一个slice是否为空需要使用len(s) == 0，而不是s == nil，因为s != nil的情况下，slice也有可能为空

### append 函数

内置函数append可以将元素追加到slice的后面，下面通过一个自己实现的appendInt方法来理解内置函数append

```go
func appendInt(x []int, y int) []int {
	var z[] int
	zlen := len(x) + 1
	if zlen < cap(x) {
		z = x[:zlen]
	}else {
		zcap := zlen
		if zcap < 2 * len(x) {
			zcap = 2 * len(x)
		}
		z = make([]int, zlen, zcap)
		copy(z, x)

	}
	z[len(x)] = y
	return z
}
func main() {
	var x, y []int
	for i := 0;i < 10; i++ {
		y = appendInt(x, i)
		fmt.Printf("%d cap = %d\t%v\n", i, cap(y), y)
		x = y
	}
}
//效果
1 cap = 2       [0 1]
2 cap = 4       [0 1 2]
3 cap = 6       [0 1 2 3]
4 cap = 6       [0 1 2 3 4]
5 cap = 10      [0 1 2 3 4 5]
6 cap = 10      [0 1 2 3 4 5 6]
7 cap = 10      [0 1 2 3 4 5 6 7]
8 cap = 10      [0 1 2 3 4 5 6 7 8]
9 cap = 18      [0 1 2 3 4 5 6 7 8 9]
```

每一次的appendInt的调用都必须检查slice是否有足够的容量来存储新的元素，如果slice容量足够，那么就会定义一个新的slice，但是该slice仍然引用原始底层数组，然后将新元素y复制到新的位置，并返回这个slice，也就是说输入参数slice和函数的返回值slice z具有相同的返回值

如果slice的容量不过容纳增长的元素，appendInt函数必须创建一个新的具有足够容量的新的底层数组来存储新的元素，然后将slice x复制到这个数组，再将新元素y追加到数组后面，返回值slice z和输入参数slice

![image-20210603103528300](http://cdn.noteblogs.cn/image-20210603103528300.png)



![image-20210603103547033](http://cdn.noteblogs.cn/image-20210603103547033.png)

再通过一个例子理解append之后的slice底层数组是否改变的问题

```go
func main() {
	s := []int{5}
	fmt.Println("cap(s) = ", cap(s), "ptr(s) = ", &s[0])


	s = append(s, 7)
	fmt.Println("cap(s) = ", cap(s), "ptr(s) = ", &s[0])

	s = append(s, 9)
	fmt.Println("cap(s) = ", cap(s), "ptr(s) = ", &s[0])

	x := append(s, 11)
	fmt.Print("cap(s) = ", cap(s), " ptr(s) = ", &s[0], " ptr(x) =", &x[0])
	fmt.Println(" s中的元素: ", s, "x中的元素: ", x)

	y := append(s, 12)
	fmt.Print("cap(s) = ", cap(s), " ptr(s) = ", &s[0], " ptr(y) =", &y[0])
	fmt.Println(" s中的元素: ", s, "y中的元素: ", x)

}
```

![image-20210603140433548](http://cdn.noteblogs.cn/image-20210603140433548.png)

- 创建s时，cap(s) == 1现在内存中的数据只有[5]
- s = append(s, 7)，按照slice的扩容机制，cap(s)翻倍，这个时候内存中的数据[5,7]，创建了一个新的底层数组
- s = append(s, 9)，同样的按照slice的扩容机制，cap(s)翻倍，内存中的数据为[5,7,9]
- x := append(s, 11)：不需要扩容，x内存中的数据为[5,7,9,11]但是s的数据为[5,7,9]，`需要注意的事情是slice x和slice s指向的底层数组是同一个数组`

- y := append(s, 12)：与上面同理