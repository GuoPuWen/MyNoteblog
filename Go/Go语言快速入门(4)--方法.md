# 方法

面向对象的封装和组合原则，需要靠内部的方法真正的实现，Go中的方法声明和普通函数的声明类似，只是在函数名前多写一个参数，这个参数把这个方法绑定到这个参数对应的类型上

```go
type Point struct {
	X, Y float64
}
//普通函数
func Distance(p, q Point) float64 {
	return math.Hypot(q.X - p.X, q.Y - p.Y)
}
//Point类型的方法
func (p Point) Distance(q Point) float64 {
	return math.Hypot(q.X - p.X, q.Y - p.Y)
}
```

- 上面两个Distance函数声明没有冲突，第一个声明包级别的函数，第二个声明类型Point的方法
- 每一个类型都有自己的命名空间，由此能够在其他不同的类型中使用名字Distance作为方法名

```go
type Path []Point	//slice类型
//计算线段相邻点的距离
func (path Path) Distance() float64 {
	sum := 0.0
	for i := range path {
		if i > 0 {
			sum += path[i - 1].Distance(path[i])
		}
	}
	return sum
}
```

- 只要类型不是指针类型或者接口类型，其他类型都可以声明方法
- `如果一个函数需要更新一个变量，或者一个实参太大希望避免复制整个实参，因此必须使用指针来传递变量的址`，习惯上，如果一个类的任何一个方法使用指针接收者，那么所有的方法都使用指针接收者

```go
func (p *Point) ScaleBy(factor float64){
	p.X *= factor
	p.Y *= factor
}
func main() {
	r1 := &Point{1, 2}
	r1.ScaleBy(2)
	fmt.Println(*r1)

	r2 := Point{1,2}
	r2ptr := &r2
	r2ptr.ScaleBy(2)
	fmt.Println(r2)

	r3 := Point{1,2}
	(&r3).ScaleBy(2)
	fmt.Println(r3)

	r4 := Point{1,2}
	r4.ScaleBy(2)
	fmt.Println(r4)	//会做一个隐式的变换
}
```

- 可以使用方法变量与方法表达式

```go
	p := Point{1,2}
	q := Point{4,6}
	distanceFromP := p.Distance	//方法变量
	fmt.Println(distanceFromP(q))
	var origin Point 	//{0,0}
	fmt.Println(distanceFromP(origin))

	distance := Point.Distance
	fmt.Println(distance(p, q))		//方法表达式
	fmt.Printf("%T\n", distance)
```

- 在GO中只有一种方式控制命名的可见性：定义的时候，首字母大写的标识符是可以从包中导出的，而首字母没有大写的则不导出

# 接口

- 在go中，接口包含两种含义：是方法的集合，同时还是一种类型，在go中这种实现是一种隐式的，就是说对于一个具体类型，不需要声明它具体实现了那些接口

```go
type Human interface {
	Say()
}
type man struct {

}
type women struct {

}

func Say(human Human)  {
	human.Say()
}
func (w *women) Say() {
	fmt.Println("women say")
}
func (m *man) Say() {
	fmt.Println("man say")
}

func main() {
	w := new(women)
	Say(w)
	m := new(man)
	Say(m)
}
```

- 对于interface{}类型不是任意类型，是属于接口类型，但是可以转为任意类型

```go
type Test struct {}

func Print(v interface{})  {
	println(v)
}

func main() {
	v := Test{}
	Print(v)
}
```

当往Print函数内部传入值的时候，Go会自动的进行类型转换，将该值转换成为接口类型的值，所有的值在运行的时候都只会有一个类型。在Go中接口有两部分组成，`一个指向该接口的具体类型的指针，一个指向该具体类型的具体数据的指针`

```go
type iface struct {
    tab  *itab
    data unsafe.Pointer
}

type eface struct {
    _type *_type
    data  unsafe.Pointer
}
```

- 将上述代码修改一下，会发现指针接收者和值类型之间的区别

```go
type Human interface {
	Say()
}
type man struct {

}
type women struct {

}

func Say(human Human)  {
	human.Say()
}
func (w *women) Say() {
	fmt.Println("women say")
}
func (m man) Say() {
	fmt.Println("man say")
}

func main() {
	w := man{}
	Say(w)
	m := women{}
	Say(m)
}
```

会报错，提示women没有实现Human接口，因为women实现Human接口定义的是指针接收者，但是在使用的时候使用的却是women的值，所以会报错，修改代码为，便可以编译通过

```go
func main() {
	w := &man{}
	Say(w)
	m := &women{}
	Say(m)
}
```

 不过有一点，对于man来说实现接口定义的是值，但是在使用的时候却是指针类型，仍然可以编译通过？需要注意的是在Go中都是值传递，尽管传入的是指针，但是可以通过该指针去找到对应的值，Go隐式的做了这个转换，也就是说`通过指针类型可以获得该类型的值，但是通过该值不一定可以找到该值被指向的指针，道理就是一个值可以被很多指针同时指向，但是一个指针只能指向一个值`



- 类型断言

```go
<目标类型>, <布尔参数> := <表达式>.(目标类型) //这种是安全的类型断言, 不会引发 panic.
<目标类型> := <表达式>.(目标类型) //这种是非安全的类型断言, 如果断言失败会引发 panic.
```

```go
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Object interface {
	Volume() float64
}

type Skin interface {
	Color() float64
}

type Cube struct {
	side float64
}

func (c *Cube) Area() float64 {
	return c.side * c.side
}

func (c *Cube) Volume() float64 {
	return c.side * c.side * c.side
}

func main() {
	var s Shape = &Cube{
		side: 3.0,
	}
	values1, ok1 := s.(Object)
	fmt.Printf("%v, %v", values1, ok1)
}
```

