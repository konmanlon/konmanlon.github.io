# 结构体



参考：

- https://golangbot.com/structs/

- https://gobyexample.com/structs
- https://go.dev/ref/spec#Struct_types



## 什么是结构体？

- Go 语言中的结构体类似于面向对象中的 “类”，但不完全是，Go语言也没打算真正实现面向对象范式。

- Go 的结构体是用户定义类型，表示字段的集合，用于将数据分组为一个单元，而不是将每个数据作为一个单独的值存放

- 每一个该结构体的实例都拥有该结构体中的所有属性



## 声明结构体

```go
type Person struct {
	firstName, lastName string
	age  int
}
```

上面的代码片段声明了一个 `Person` 类型的命名结构体，其中包含 `firstName`, `lastName` , `age` 三个字段（属性或成员）；

`Person` 是一个标识符，指代 `struct {}`

相同的字段类型可以写在同一行，虽然看起来更加紧凑，但显得不够明确。



## 初始化结构体

```go
package main

import "fmt"

type Person struct {
	firstName string
	lastName  string
	age       int
}

func main() {

    // 使用 var 声明一个 Person 类型的 p1 实例（没有赋值），零值可用
	var p1 Person

    // 字面量创建实例，零值初始化
	p2 := Person{}

    // 字面量定义字段并赋值，字面量初始化
    // 指定字段名赋值与顺序无关，且字段可缺省，缺省字段使用零值
	p3 := Person{
		firstName: "Cruise",
		lastName:  "Tom",
		// age:       61,
	}

    // 不指定字段名赋值时，必须按照字段顺序全部赋值，不可缺省
	p4 := Person{"Cruise", "Tom", 61}

	fmt.Printf("%v\n%v\n%v\n%v", p1, p2, p3, p4)

}
```

输出结果：

```tex
{  0}
{  0}
{Cruise Tom 0}
{Cruise Tom 61}
```



## 访问结构体

可见性（导出）：

- Go package 的顶层代码中，首字母大写的标识符，跨 package 可见（导出），否则只能本包内可见
- 导出的结构体，package 内外皆可见，同时，导出的结构体中的成员（属性、方法）要在包外也可
  见，则也需首字母大写
- 使用 `.` 运算符访问结构体中的各个字段



示例：

```go
package main

import "fmt"

type Person struct {
	firstName string
	lastName  string
	age       int
}

func main() {

	p4 := Person{"Cruise", "Tom", 61}

    // 访问 Person 中的 firstName 属性
	fmt.Println(p4.firstName)

}
```



## 修改结构体

示例：

```go
package main

import "fmt"

type Person struct {
	firstName string
	lastName  string
	age       int
}

func main() {

	p1 := Person{"Cruise", "Tom", 61}

	fmt.Println(p1.lastName)

    // 给字段重新赋值
	// p1.lastName = "Jerry"
    p1.lastName, p1.age = "Jerry", 50

	fmt.Println(p1.lastName)

}
```



## 结构体方法

结构体的方法、属性，称为成员。方法也称为成员方法，属性也称为成员属性。



示例：

```go
package main

import "fmt"

type Person struct {
	firstName string
	lastName  string
	age       int
}

// 使用普通函数获取实例的属性，该函数接收一个 Person 类型的参数
// 该普通函数跟结构体无任何关系，只是参数是 Person 类型
func getLastName(l Person) string {
	return l.lastName
}

// 使用结构体方法获取实例的属性，结构体方法本质上还是函数，Go 的语法糖
// 在 func 关键字与函数标识符的中间定义一个 Person 类型的 Receiver (l)
// 使该方法与 Person 类型的结构体关联，且该方法只能被 Person 结构体调用
func (l Person) getLastName() string {
	return l.lastName
}

func main() {

	// 实例化一个 p1 对象
	p1 := Person{"Cruise", "Tom", 61}

    // 调用普通函数并传递参数
	fmt.Println(getLastName(p1))

    // 通过 Person 的实例(p1)调用方法，方法的 Receiver 为 p1
	fmt.Println(p1.getLastName())

}
```



### Receiver

> 什么是 Receiver?
>
> - Go 中的接收者是一个特殊的参数，在调用方法时传递给方法。接收者参数是对调用该方法的结构体的引用。这允许该方法访问和修改结构体中的字段。
>
> - Go 中有两种类型的接收者：值接收者和指针接收者。值接收者是对其调用方法的结构体的引用。指针接收者是指向正在调用其方法的结构体的指针。

一般来说，如果不需要修改结构体，则应使用值接收者；如果需要修改结构体，则应使用指针接收者。

指针接收者效率更高，减少了值的复制和内存占用。



两种类型的接收者比较： 

| Feature                   | Value Receiver                           | Pointer Receiver                   |
| :------------------------ | :--------------------------------------- | :--------------------------------- |
| Type                      | A reference to the struct                | A pointer to the struct            |
| How it is passed          | By value                                 | By pointer                         |
| Can it modify the struct? | Yes                                      | Yes, but it is more efficient      |
| When to use it            | When you don't need to modify the struct | When you need to modify the struct |



值接收者示例：

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
}

// 值接收者
func (p1 Person) getName() string {
	fmt.Printf("%p %[1]v\n", &p1)
	return p1.name
}

func (p1 Person) getAge() int {
	fmt.Printf("%p %[1]v\n", &p1)
	return p1.age
}

func (p1 Person) setName(x string) {
	p1.name = x
	fmt.Printf("%p %[1]v\n", &p1)
}

func (p1 Person) setAge(x int) {
	p1.age = x
	fmt.Printf("%p %[1]v\n", &p1)
}

func main() {

	p1 := Person{"tom", 60}
	fmt.Printf("%p %[1]v\n", &p1)

	fmt.Println(
		p1.getName(),
		p1.getAge(),
	)

	p1.setName("jerry")
	p1.setAge(50)

}

```

输出结果：

- 每次都是完整的值拷贝，在内存中开辟了新地址
- 操作的是副本，因此初始实例不会被修改

```go
0xc000010030 &{tom 60}
0xc000010060 &{tom 60}
0xc000010090 &{tom 60}
tom 60
0xc0000100c0 &{jerry 60}
0xc0000100f0 &{tom 50}
```



指针接收者示例：

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
}

// 指针接收者
func (p1 *Person) getName() string {
	fmt.Printf("%p %[1]v\n", p1)
	return p1.name
}

func (p1 *Person) getAge() int {
	fmt.Printf("%p %[1]v\n", p1)
	return p1.age
}

func (p1 *Person) setName(x string) {
	p1.name = x
	fmt.Printf("%p %[1]v\n", p1)
}

func (p1 *Person) setAge(x int) {
	p1.age = x
	fmt.Printf("%p %[1]v\n", p1)
}

func main() {

	p1 := Person{"tom", 60}
	fmt.Printf("%p %[1]v\n", &p1)

    // 打印函数的返回值
	fmt.Println(
		p1.getName(),
		p1.getAge(),
	)

	p1.setName("jerry")
	p1.setAge(50)

}

```

输出结果：

- 只拷贝了实例的地址，不拷贝实例的内容，因此减少了值复制，降低了内存开销
- 操作的是同一个内存的同一个实例，因此初始实例会被修改

```go
0xc00019c018 &{tom 60}
0xc00019c018 &{tom 60}
0xc00019c018 &{tom 60}
tom 60
0xc00019c018 &{jerry 60}
0xc00019c018 &{jerry 50}
```



> Go 语言中都是值拷贝，指针类型只拷贝地址



## 结构体指针

结构体是值类型，使用的是值拷贝。传参或返回值如果使用结构体实例，将产生很多副本，使用指针可减少副本。



示例 1：

```go
package main

import "fmt"

type Person struct {
	firstName string
	lastName  string
	age       int
}

func main() {
	p1 := &Person{"Cruise", "Tom", 61}
    
    // new() 创建该结构体的零值实例，并返回该零值实例的指针
	p2 := new(Person)

	p3 := Person{"Cruise", "Tom", 61}

	p4 := &p3

	p5 := p4

    // 显示取消引用指针
	fmt.Printf("%v\n", (*p1).age)
    // Go 语法糖，隐式取消了引用指针
	fmt.Printf("%v\n\n", p1.age)

    // 由于 new() 返回的就是指针类型，再使用 & 对 p2 取地址就成了二级指针
	fmt.Printf("%v %[1]T\n", &p2)
    // 打印 p2 的地址和类型
	fmt.Printf("%p %[1]T\n", p2)
    // 打印 p2 的值
	fmt.Printf("%v %[1]T\n", *p2)
	fmt.Printf("%v %[1]T\n\n", (*p2))
    
    // 通过指针修改属性
    p2.age = 50
	fmt.Printf("%p %v\n\n", p2, *p2)

	fmt.Printf("%p\n%p\n%p\n", &p3, p4, p5)
}
```

输出结果：

```tex
61
61

0xc0001a4018 **main.Person
0xc000092150 *main.Person
{  0} main.Person
{  0} main.Person

0xc000108150 {  50}

0xc000194180
0xc000194180
0xc000194180
```



示例 2：

```go
package main

import "fmt"

type Person struct {
	firstName string
	lastName  string
	age       int
}

func test(p *Person) *Person {
	p.age = 51
	fmt.Printf("%p %v\n", p, (*p).age)
	return p
}

func main() {

	p1 := &Person{"Cruise", "Tom", 61}
	fmt.Printf("%p %v\n", p1, (*p1).age)

	p2 := test(p1)
    
	p2.age = 41
	fmt.Printf("%p %v\n", p2, (*p2).age)

}
```

输出结果：

```go
0xc000108150 61
0xc000108150 51
0xc000108150 41
```





> 什么是指针？
>
> - 在 Go 中，指针是一个存储另一个变量地址的变量。变量的地址是它在内存中的位置。创建指针时，实质上是创建对另一个变量的引用。
> - `*`运算符用于取消引用指针。可以使用`*`运算符来获取指针指向的值。
> - `&`运算符用于获取变量的地址。可以使用`&`运算符创建指向变量的指针。



## 匿名结构体

可以在不创建新数据类型的情况下声明结构体（不使用 type 定义）。这些类型的结构体称为匿名结构体。

使用 type 定义的命名结构体可以比作一个模板，可以实例化 N 个结构体的实例，而匿名结构体是一次性的。

type 标识符指向的是一个类型，匿名结构体的标识符指向的是一个实例。



示例：

```go
package main

import "fmt"

func main() {
    // 定义匿名结构体变量，而不是使用 type 定义新的结构体类型
    
    // 该方式得到的是一个结构体实例，Person1 标识符后面的类型零值实例化后给 Person1
	var Person1 struct {
		x int
		y int
	}
	fmt.Printf("%#v\n", Person1)

    // Person2 的定义与上面的方式等价，零值初始化
	var Person2 = struct {
		x, y int
	}{}
	fmt.Printf("%#v\n", Person2)

    // Person3 短格式定义，字面量初始化
	Person3 := struct {
		x, y int
	}{1, 2}
	fmt.Printf("%#v\n", Person3)
}
```

输出结果：

```go
struct { x int; y int }{x:0, y:0}
struct { x int; y int }{x:0, y:0}
struct { x int; y int }{x:1, y:2}
```



## 匿名成员

可以创建仅包含字段类型而不包含字段名称结构体。此类字段称为匿名字段。

```go
package main

import "fmt"

func main() {
    
    // 默认情况下匿名字段会采用其类型的名称作为标识，因此同一数据类型的字段中只能有一个匿名字段
	type Person struct {
		string
		int
		bool
	}

	p1 := Person{
		string: "a",
		int:    50,
		bool:   true,
	}

	fmt.Printf("%v\n", p1)
}
```



## 构造函数

Go语言并没有从语言层面为结构体提供什么构造器，但是有时候可以通过一个函数为结构体初始化提供属性值，从而方便得到一个结构体实例。习惯上，函数命名为 NewXxx 的形式。



当结构体的成员有大量属性时，用普通方式实例化比较繁琐，因此可以构造一个结构体函数并为一些可选字段赋初值等（Go 的函数不支持缺省值），使在调用函数时提供少量必须的参数即可完成实例化。



示例：

```go
package main

import "fmt"

type Hook struct {
	project string
	env     string
	path    string
}

// 定义构造结构体的函数，接收两个参数，并返回结构体实例
func NewHook(project, env string) *Hook {
	return &Hook{project, env, "/v1"}
}

func main() {
	c1 := Hook{"p2", "dev", "/v2"}
	fmt.Println(c1)

	c2 := NewHook("p1", "dev")
	fmt.Println(*c2)
}
```



## 结构体嵌套

结构体中的字段也可以是一个结构体，这种情况称为结构体嵌套；嵌套的结构体类似面向对象语言中继承的概念，如子结构体可以继承父结构体中的属性。



示例：

```go
package main

import "fmt"

// 定义父结构体，通常抽象范围比较大，例如作为不同子结构体共同依赖的属性
type Address struct {
	city  string
	state string
}

// 定义子结构体，通常抽象的实例更具体
type Person struct {
	name    string
	age     int
	address Address  /** 将父结构体嵌入子结构体，以让子结构体继承父结构体的属性 **/
}

func main() {
    // 字面量实例化
	p1 := Person{
		name: "tom",
		age:  60,
		address: Address{
			city:  "Los Angeles",
			state: "Chesterfield Square",
		},
	}
    // 访问子结构体本身的字段
	fmt.Println(p1.name)
    // 访问“继承”父结构体中的字段
	fmt.Println(p1.address.city)
}

```



上面的例子是在命名字段中嵌入结构体，因此在访问父结构体中的字段时必须先通过子结构体中的字段去访问；而使用匿名字段的方式嵌入结构体时，可以通过实例直接访问父结构体的字段，在实际写代码的过程中更多的也是采用这种简便的方式，如下示例：

```go
package main

import "fmt"

type Address struct {
	city  string
	state string
}

type Person struct {
	name    string
	age     int
	Address  /** 匿名嵌入 **/
}

func main() {
	p1 := Person{
		name: "tom",
		age:  60,
        // 匿名字段的结构体字面量初始化，默认情况下匿名字段会采用其类型的名称作为标识
		Address: Address{
			city:  "Los Angeles",
			state: "Chesterfield Square",
		},
	}
	fmt.Println(p1.name)
    // 修改父结构体的字段
    p1.state = "Wilshire Blvd"
    // 通过实例直接访问父结构体的字段，匿名成员的语法糖
	fmt.Println(p1.state, p1.city)
    // 完整的属性修改和访问
    p1.Address.state = "Chesterfield Square"
    fmt.Println(p1.Address.state)
}
```



## 深浅拷贝

