# Interfaces



参考资料：

- https://golangbot.com/interfaces-part-1/
- https://golangbot.com/interfaces-part-2/
- https://go.dev/ref/spec#Interface_types

## 什么是 interfaces？

在 Go 中接口是一组方法签名，可以将接口看作为是一组行为规范（约束）的集合。

要实现一个接口就必须把接口中声明的方法全部都实现



示例：

```go
// A simple File interface.
interface {
	Read([]byte) (int, error)
	Write([]byte) (int, error)
	Close() error
}
```

> - 接口命名习惯在接口名后面加上er后缀，方法名称必须是唯一的，且不能为空
> - 参数列表、返回值列表参数名可以不写
> - 如果要在包外使用接口，接口名应该首字母大写，方法要在包外使用，方法名首字母也要大写
> - 接口中的方法应该设计合理，不要太多



## 声明和实现接口

示例：

```go
package main

import (
	"fmt"
)

// 声明一个接口
type Create interface {
	getMake() string
	getModel() string
	getColor() string
}

// 声明一个结构体，并实现 Create 接口
type Car struct {
	make  string
	model string
	color string
}

func (c *Car) getMake() string {
	fmt.Printf("%p\n", c)
	return c.make
}

func (c *Car) getModel() string {
	fmt.Printf("%p\n", c)
	return c.model
}

func (c *Car) getColor() string {
	fmt.Printf("%p\n", c)
	return c.color
}

// 创建构造函数返回一个接口，返回的实例必须已实现该接口，Go 语言推荐使用这种方式
func newCar(s1, s2, s3 string) Create {
	return &Car{s1, s2, s3}
}

func main() {
	c := Car{"tesla", "x", "black"}
	fmt.Printf("%p\n", &c)
	fmt.Printf("%v %[1]T\n", c.getMake())
	fmt.Printf("%v %[1]T\n", c.getModel())
	fmt.Printf("%v %[1]T\n", c.getColor())

	fmt.Println("~~~~~~~~~~~~~~~~~~~~~~~~~")
	// method is pointer receiver
    // 使用接口：接口中的方法有指针接收者，因此赋值必须是实例的地址
	var d Create = &c
	fmt.Println(d.getMake())
	fmt.Println(d.getModel())
	fmt.Println(d.getColor())

	fmt.Printf("%T %[1]v\n", d)

    // 调用构造函数创建 Create 接口的实例
    // 等价于该写法： var e Create = &c
	e := newCar("tesla", "y", "black")
	fmt.Println(e.getMake())
	fmt.Println(e.getModel())
	fmt.Println(e.getColor())

	fmt.Printf("%T %[1]v\n", e)
}

```

> - 一种类型可以实现多个接口

## 嵌入式接口

如果接口比较复杂，可以定义多个小接口，再组合成大接口。



示例：

```go
type Reader interface {
	Read(p []byte) (n int, err error)
	Close() error
}

type Writer interface {
	Write(p []byte) (n int, err error)
	Close() error
}

// 嵌入接口：使 ReadWriter 接口包含 Reader 和 Writer 接口中的方法
type ReadWriter interface {
	Reader
	Writer
}

// 嵌入接口时，相同名称的方法，签名必须也必须一致
type ReadCloser interface { 
	Reader 
	Close() // 非法：Reader.Close 和 Close 的签名不同
}
```



## 空接口

具有零个方法的接口称为空接口，由于空接口有零个方法，因此所有类型都实现了空接口，且类型的值都可以看做是空接口类型，所以空接口可以用于接收任意类型的实例。



空接口可使用以下两种形式表示：

- `interface{}`
- `any` (`interface{}`的别名：`type any = interface{}`)



示例：

```go
package main

import (
	"fmt"
)

func main() {
    // 定义空接口变量
	var a interface{}
	fmt.Printf("%T %[1]v\n", a)

    // b 是 int 类型的实例，也是空接口类型的实例
	var b = 100
    // c 是 string 类型的实例，也是空接口类型的实例
	var c = "hello"

    // a 是空接口类型的变量，因此赋任意类型的值
    // 但打印 a 的值类型时，是其值本身的类型，而不是空接口类型
	a = b // var a interface{} = b
	fmt.Printf("%T %[1]v\n", a)
	a = c // var a any = c
	fmt.Printf("%T %[1]v\n", a)

    // 三种空接口类型的空切片等价写法，空接口类型的切片：[]interface{}
	d1 := []any{}
    d2 := []interface{}{}
    var d3 []interface{}
    fmt.Printf("%T %[1]v\n%T %[2]v\n%T %[3]v\n", d1, d2, d3)
    
    // 字面量赋值
	var e1 any = 1
	var e2 interface{} = 2
	fmt.Printf("%T %[1]v\n%T %[2]v\n", e1, e2)
	e3 := []any{1, "a"}
	var e4 = []interface{}{1, "a"}
	fmt.Printf("%T %[1]v\n%T %[2]v\n", e3, e4)
    
    // 空接口类型的切片中的元素可以是任意类型
	s1 := make([]interface{}, 0)
	s1 = append(s1, 1)
	s1 = append(s1, 'a')
	s1 = append(s1, "b")
	s1 = append(s1, []int{})
	s1 = append(s1, map[string]int{})
	fmt.Printf("%T %[1]v\n", s1)

	s2 := make([]any, 5)
	s2[0] = 1
	s2[1] = 'a'
	s2[2] = "b"
	s2[3] = []int{}
	s2[4] = map[string]int{}
	fmt.Printf("%T %[1]v\n", s2)
}

```

输出结果

```go
<nil> <nil>
int 100
string hello
[]interface {} []
[]interface {} []
[]interface {} []
int 1
int 2
[]interface {} [1 a]
[]interface {} [1 a]
[]interface {} [1 97 b [] map[]]
[]interface {} [1 97 b [] map[]]
```



> - 空接口零值为`nil`
> - 对空接口类型的实例取值类型时，得到是值原始的类型而非空接口类型

## 接口类型断言

类型断言用于获取空接口值的原始类型



示例：

```go
package main

import (
	"fmt"
)

func assert1(i interface{}) {
    // i.(<type>) 对空接口类型变量进行断言，s 接收断言结果
    // 此语法断言成功会返回变量值，断言失败会 panic
	s := i.(int)
	fmt.Println(s)
}

func assert2(i interface{}) {
    // 此语法断言成功会返回变量值和 true，断言失败会返回断言类型(int)的零值和 false，程序不会 panic
	v, ok := i.(int)
	fmt.Println(v, ok)
}

func main() {
	var a interface{} = 100
	assert1(a)

	var b interface{} = 200
	assert2(b)

	var c interface{} = "hello"
	assert2(c)
}
```

输出结果：

```tex
100
200 true
0 false
```



### Type switch

当接口类型较多时，可以使用 type switch 进行断言，`i.(type)` 是固定语法，只能在 switch 中使用。



示例：

```go
package main

import (
	"fmt"
)

func findType(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("I am a int and my value is %d\n", v)
	case string:
		fmt.Printf("I am a string and my value is %s\n", v)
	default:
		fmt.Printf("Unknown type\n")
	}
}

func main() {
	findType(1)
	findType("hello")
	findType([]int{1})
}
```

输出结果：

```tex
I am a int and my value is 1
I am a string and my value is hello
Unknown type
```



示例：

```go
package main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() {  
    fmt.Printf("%s is %d years old", p.name, p.age)
}

func findType(i interface{}) {  
    switch v := i.(type) {
    // 如果 Person 类型实现了 Describer 接口，则调用接口中的 Describe() 方法
    case Describer:
        v.Describe()
    default:
        fmt.Printf("unknown type\n")
    }
}

func main() {  
    findType("Naveen")
    p := Person{
        name: "Naveen R",
        age:  25,
    }
    findType(p)
}
```


