# map

map是散列表的引用，map的类型是map[k]v，map中的键都拥有相同的数据类型，值也拥有相同的数据类型，键和值的数据类型不一定要一样，`键的类型k，必须是可以通过操作符==类进行比较的数据类型`，基本使用如下

```go
func main() {
	ages := make(map[string]int)
	ages["alice"] = 31
	ages["charlie"] = 34
	//删除键
	delete(ages, "alice")
	//键不在map中 下面操作也是安全的
	ages["bob"]= ages["bob"] + 1
	//循环获取键和值
	for name, age := range ages {
		fmt.Printf("%s\t%d\n", name, age)
	}

}
```

- map中的元素迭代顺序是不一样的和HashMap是一样的
- map类型的零值是nil，也就是没有引用任何散列表，设置元素之前，必须初始化map
- 通过下标访问map中的元素总会有值，如果map存在该键，按摩得到该键对应的值，如果不存在将得到map值类型的零值。可以使用下面的方法判断map键是否存在

```go
//判断是否存在元素
//输出两个值 第一个值为键 第二个值为布尔值，用来报告该元素是否存在
age, ok := ages["bob"]
if !ok {
    //不是字典中的键 进行操作
}
//还可以这样写
if age, ok := ages["bob"]; !ok {//操作}
```

# 结构体

结构体是将零个或者多个任意类型的变量组合在一起的聚合数据类型，每一个变量都是结构体的成员

```go
//定义一个结构体
type Employee struct {
	ID 			int
	Name		string
	Address		string
	DoB			time.Time
	Position	string
	Salary		int
	ManagerID	int
}

func main() {
	var dilbert Employee
	//给结构体变量赋值
	dilbert.Salary -= 5000
	//获取成员遍历的地址 然后赋值
	position := &dilbert.Position
	*position = "Senior" + *position
	//也可以获取结构体指针 然后赋值
	var employeeOfTheMonth *Employee = &dilbert
	employeeOfTheMonth.Position += "(proactive team player)"
	fmt.Println(dilbert.Position)
}
```

- 结构体对于相同的类型可以连续的写在一行上，但是顺序对于结构体很重要，也就是说如果顺序不一样，将得到两个不同的结构体
- 如果一个结构体的成员变量名称是首字母大写的，那么这个变量是可导出的
- 如果结构体的所有成员变量都可以比较，那么这个结构体就是可以比较的，既然是可比较的类型，所以可以作为map的键

```go
p := Point{1,2}
q := Point{2, 1}
fmt.Println(p == q)
```

- 存在结构体嵌套机制，可以将一个命名结构体当做另外一个结构体类型的匿名成员使用，并且可以使用简单的表达式代表连续的成员

```go
type Point struct {
	X, Y 		int
}

type Circle struct {
	Point
	Radius		int
}

type Wheel struct {
	Circle
	Spokes		int
}

func main() {
	//var dilbert Employee
	var q Wheel
	//初始化方式
	q = Wheel{
		Circle : Circle{
			Point: Point{X: 8, Y: 8},
			Radius: 5,
		},
		Spokes: 20,
	}
	fmt.Println(q)
}
```

