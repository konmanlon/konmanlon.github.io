# Go 编程



## 变量

变量交换

```go
a, b = b, a

// 借助中间变量
tmp := a
a = b
b = tmp
```



## 函数

理解函数：

- 函数可以有返回值，也可以没有返回值

- 函数可以作为参数传递给函数，也可以作为返回值返回



定义函数：

1. func 声明

2. 函数名也是函数标识符，需要符合名称规范

3. 函数名后面的小括号定义形参（Go 中不支持设置默认值），当多个形参类型连续一样时，可只在最后一个形参后定义类型

4. 大括号中定义函数体



调用函数：

- 函数名加小括号，如有参数需要在括号中传递



示例：

```go
packcge main

// 无参数，无返回值
func main() {...}

// 有参数(形参)，多个参数使用逗号分隔，调用该时传递的值称为实参
func sayHi(name, string) {
	fmt.println(name)
}

// 有参数，有返回值，(int) 定义返回值的类型
// 当定义了返回的时候，函数体中的每个分支都必须要有返回值，否则编译不通过
// return 语句之后的代码都不会再执行, 但 return 可以返回多个不同类型值
// 函数有返回值的时候，即可在调用该函数时接收返回结果
// 当多个形参类型连续一样时，可只在最后一个形参后定义类型 func sum(a, b, c int) {...}
func sum(a int, b int) (int) {
	return a + b
}

// 可变参数, x 是可变参数，可以有多个值，是切片类型 []int
// 可变参数类型必须是一致的
// 一个函数中只能有一个可变参数，且可变参数只能放在最后
// 如果要将该 x 可变参数传递给其他函数时，可以使用 ... 符号将切片解包成单个元素
func param(a int, b int, x ...int) int {
	// 打印 x 的类型
	fmt.Printf("%T", x)
	// 将 x 解包传递给 test()
	return test(x...)
}
```



示例：return

```go
package main

import "fmt"

func sum(a int, b int) int {
	return a + b
}

func main() {
	num := sum(10, 20)
	fmt.Println(num)
}
```



示例：遍历可变参数

```go
package main

import "fmt"

func param(a, b int, x ...int) int {
	total := a + b
	// _ 空白符接收索引, range 遍历 []int
	for _, v := range x {
		total += v
	}
	return total
}

func main() {
	sum := param(1, 2, 3, 4, 5)
	fmt.Println(sum)
}
```



示例：return 返回多个值，值的类型也可以不相同，且也可以调用变量返回

```go
package main

import "fmt"

func param(a, b int) (int, int, int, int) {
	return a + b, a - b, a * b, a / b
}

// 调用变量返回，没有定义变量时返回值默认为 0, 返回值连续类型一致可省略，在最后一个返回值定义类型即可
func param1(a, b int) (num1, num2, num3, num4, num5 int) {
	num1 = a + b
	num2 = a - b
	num3 = a * b
	num4 = a / b
	return
}

func main() {
	// 直接打印所有返回值
	fmt.Println(param(10, 5))
	
	// 定义变量来接收返回值（变量数量与返回值数量相同），不需要的返回值可以使用 _ 空白符接收
	a, b, c, _ := param(10, 5)
	fmt.Println(a, b, c)
	
	// 调用 param1
	fmt.Println(param1(6, 4))
}

// 15 5 50 2
// 15 5 50
// 10 2 24 1 0
```



## 递归

理解递归：

- 直接或间接的调用自己，解决重复的分支问题（如：一个问题由多个相同的小问题组成）

- 递归必须有终止条件，否则是死循环



## 函数的类型

理解函数类型：

- 函数参数、参数数量、参数类型、返回值，为函数的签名

- 当将一个函数赋值给函数类型的变量时，函数的签名必须一致

```go
func sum(a int, b int) int {
	return a + b
}

func main() {
	// sum 的类型需要与 f 一致才能赋值
	var f func(int, int) int = sum
	
	// 调用 f
	fmt.Println(f(1, 2))
}
```



示例：函数形参

```go
func f1(callback func(...string), args ...string) {
    // 调用 callback 函数，并将 args 切片解包传递给 callback 函数
    callback(args...)
}

// list 函数的签名与 callback 的签名一致
func list(args ...string) {
    // i 接收索引，v 接收元素
    for i, v := range args {
        fmt.Println(i, ":", v)
    }
}

func main() {
    // 调用 f1 函数，并传递 list 函数给 f1，f1 中的 callback 就是 list 函数
    // 用 list 函数处理 args
    f1(list, "a", "b", "c")
}

// 0 : a
// 1 : b
// 2 : c
```



## 匿名函数

理解匿名函数：

- 没有名字的函数直接使用()调用，或将函数赋值给了一个变量通过变量来调用函数

- 匿名函数同样可作为参数传递给其他函数

```go
package main

import "fmt"

func main() {
	// 将函数赋值给变量
	sayHello := func(name string) {
		fmt.Println("Hello", name)
	}
	// 通过变量调用函数
	sayHello("World !")
	
	// 定义函数并直接调用（只能调用一次）
	func(name string) {
		fmt.Println("Hello", name)
	}("Vincent")
}
```



## 闭包

理解闭包：

- 闭包（Closure）是一种特殊的函数，它可以访问定义在它外部作用域的变量。

- 当一个函数在其定义范围之外被调用时，它可以访问并操作定义在其范围之外的变量。这些变量可以是其定义范围之外的函数参数、全局变量，或是其他函数的局部变量。当这个函数被调用时，它会创建一个闭包，将它所引用的变量的值保存在闭包中，以便在以后的调用中使用。



示例：类似匿名函数，可以用于构造函数

```go
package main

import "fmt"

func main() {
	// 定义一个函数接收 n 参数，并返回一个函数类型（函数值为 int）
	f1 := func(n int) func(int) int {
		// 返回的函数需要与 f1 返回的函数类型一致
		return func(x int) int {
			// 调用作用域外部的变量 n
			return n + x
		}
	}
	// 调用 f1 传递参数 10，会返回一个函数，然后直接(8)调用返回的函数，类似匿名函数的直接调用
	fmt.Println(f1(10)(8))

	// 用变量接收 f1 返回的新函数
	f2 := f1(10)
	// 再调用 f2
	fmt.Println(f2(8))
}

// 18
// 18
```



## 包管理与使用

包外可见性：

- 通过标识符首字母决定，首字母小写不能在包外访问，首字母大写可以



## 结构体

定义结构体：

定义结构体主要是通过 type 关键字和 struct 关键字。首先，你需要先使用 type 关键字为你的新类型命名，然后使用 struct 关键字来定义结构体的字段。

每个字段需要一个名称和一个类型。你也可以为字段添加一个可选的标签（Tag），以支持一些库，如 json 库，这可以让你在处理 JSON 格式的数据时更加灵活。

```go
type Person struct {
    Name    string `json:"name"`
    Age     int    `json:"age"`
    Address string `json:"address"`
}
```

在这个示例中，我们定义了一个名为 Person 的结构体，它包含了 Name，Age 和 Address 这三个字段。注意我们在每个字段后面都添加了一个标签，这可以帮助 json 库正确地序列化和反序列化这个结构体。



结构体的名称和属性首字母小写在包外都不可访问

定义结构体，结构体和属性首字母大写，使其包外可见

```go
type Config struct {
	Url     string
	Path    string
	Env     string
	Version int
}
```



## 结构体标签

在Go语言中，我们可以使用json标签为结构体的字段提供额外的元数据，以控制JSON序列化和反序列化的行为。

> 序列化：
>
> - 将数据结构转换为JSON格式的字符串
>
> 反序列化：
>
> - 将JSON格式的字符串转换回数据结构



示例：

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	Name  string `json:"name"`
	Age   int    `json:"age"`
	Email string `json:"email,omitempty"`
}

func main() {
	person := Person{
		Name:  "John Doe",
		Age:   30,
		Email: "johndoe@example.com",
	}

	// 将结构体转换为JSON字符串
	jsonData, err := json.Marshal(person)
	if err != nil {
		fmt.Println("JSON serialization error:", err)
		return
	}
	fmt.Println(string(jsonData))

	// 将JSON字符串反序列化为结构体
	var decodedPerson Person
	err = json.Unmarshal(jsonData, &decodedPerson)
	if err != nil {
		fmt.Println("JSON deserialization error:", err)
		return
	}
	fmt.Printf("%+v\n", decodedPerson)
}
```



注意：

- `json:"email,omitempty"`中的`omitempty`选项表示当`Email`字段为空时，不生成对应的JSON键值对。

- 在序列化时，字段名将根据`json`标签的内容进行转换。在上面的示例中，`Name`字段将被序列化为`"name"`，Age字段将被序列化为`"age"`。

- 在反序列化时，JSON键的名称将与json标签的内容匹配。如果标签未指定或为空，Go将使用字段的原始名称。



通过嵌套结构体，我们可以构建更复杂的JSON对象，该对象可以包含嵌套的子对象或子数组。这使得在Go语言中处理具有层次结构的JSON数据变得更加方便。在反序列化时，同样可以通过嵌套结构体来解析嵌套的JSON对象。

示例：

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Address struct {
	Street  string `json:"street"`
	City    string `json:"city"`
	Country string `json:"country"`
}

type Person struct {
	Name    string  `json:"name"`
	Age     int     `json:"age"`
	Address Address `json:"address"`
}

func main() {
	person := Person{
		Name: "John Doe",
		Age:  30,
		Address: Address{
			Street:  "123 Main St",
			City:    "New York",
			Country: "USA",
		},
	}

	jsonData, err := json.Marshal(person)
	if err != nil {
		fmt.Println("JSON serialization error:", err)
		return
	}

	fmt.Println(string(jsonData))
}
```



## 初始化结构体

字面量初始化：

```go
type Person struct {
	Name string
	Age  int
}

func main() {
	// 使用字面量初始化结构体
	person := Person{
		Name: "John Doe",
		Age:  30,
	}

	fmt.Printf("%+v\n", person)
}
```

在此示例中，我们通过提供字段名和对应的值来初始化结构体。这是一种直观且易读的方式，可读性较高。



零值初始化：

```go
type Person struct {
	Name string
	Age  int
}

func main() {
	// 使用零值初始化结构体
	var person Person

	fmt.Printf("%+v\n", person)
}
```

在此示例中，我们使用var关键字创建了一个零值的结构体。这将为每个字段分配其类型的零值（例如，字符串字段为空字符串，整数字段为0）。这种方式适用于需要在稍后的代码中逐渐填充结构体字段的情况。



指针初始化：

```go
type Person struct {
	Name string
	Age  int
}

func main() {
	// 使用指针初始化结构体
	person := &Person{
		Name: "John Doe",
		Age:  30,
	}

	fmt.Printf("%+v\n", person)
}
```



## 方法

特定类型的函数，函数中需要定义一个接收者



定义方法：

如前面所述，方法是关联到特定类型的函数，它有一个特殊的参数，称为接收者。方法的定义和普通函数的定义很像，只不过在函数名前面添加了一个接收者。

接收者可以是值接收者，也可以是指针接收者，这取决于你希望如何修改接收者。一般来说，如果你需要修改接收者的字段，或者你的接收者是一个大型结构，你应该使用指针接收者。

```go
type Person struct {
    Name string
    Age  int
}

func (p *Person) SetAge(age int) {
    p.Age = age
}

func (p Person) GetName() string {
    return p.Name
}
```



## 理解结构体与方法

在Go中，并没有类（class）的概念，它使用结构体（Struct）来解决问题。

结构体（Struct）是将零个或多个任意类型的命名值聚合成单个实体（即结构体的一个实例）的复合类型。每个值都称为结构体的成员。

Go语言中的方法（Method）是一种特殊类型的函数，它是定义在某种类型（可以是结构体类型，也可以是其他任何类型）上的。方法的语法和函数的语法几乎一样，它们之间的主要区别在于方法定义中的接收者部分。



示例：

```go
// 定义一个叫做 Person 的结构体
type Person struct {
    Name string
    Age  int
}

// 为 Person 结构体定义一个叫做 Greet 的方法
func (p Person) Greet() {
    fmt.Printf("Hello, my name is %s, and I'm %d years old.\n", p.Name, p.Age)
}

func main() {
    // 创建一个 Person 结构体的实例
    var person = Person{
        Name: "John",
        Age:  22,
    }
    
    // 调用 person 的 Greet 方法
    person.Greet()
}
```

在这个示例中，我们定义了一个叫做`Person`的结构体，它有两个字段：`Name`和`Age`。然后我们定义了一个名为`Greet`的方法，这个方法的接收者（即调用这个方法的具体实例）是`Person`类型的。这个`Greet`方法在被调用时，会输出一条含有其`Name`和`Age`的欢迎消息。



理解接收者：

在Go中，方法是附加到特定类型的函数。这个特定类型称为接收者类型，而这个类型的实例（值或者指针）就是接收者。

方法接收者在方法声明的函数名前面，定义了哪种类型可以使用该方法。接收者可以是值接收者，也可以是指针接收者。



值接收者: 使用值接收者定义的方法，接收者是该类型的一个副本。在方法内部对接收者做的任何修改，都不会影响原来的值。

```go
type MyType struct {
    A int
}

func (m MyType) SetValue(val int) {
    m.A = val
}

func main() {
    var m MyType
    m.SetValue(5)
    fmt.Println(m.A)  // 输出 0，因为 m 在 SetValue 方法中是一个副本，方法中的修改不会影响 m
}
```

指针接收者: 使用指针接收者定义的方法，接收者是一个指向原值的指针。在方法内部对接收者的任何修改，都会直接影响原来的值。

```go
type MyType struct {
    A int
}

func (m *MyType) SetValue(val int) {
    m.A = val
}

func main() {
    var m MyType
    m.SetValue(5)
    fmt.Println(m.A)  // 输出 5，因为 m 在 SetValue 方法中是一个指向原值的指针，方法中的修改会直接影响 m
}

```



在 Go 语言中定义方法时，必须在函数名前面定义一个接收者。这是因为接收者指定了方法绑定到的特定类型，定义了哪种类型的变量可以调用该方法。这样的设计使得方法能够明确地和某个类型关联起来。

接收者变量是一种特殊的函数参数，它在方法声明中的函数名之前。你可以把它理解为“拥有”或“具有”此方法的对象或结构。

```go
type MyType struct {
    A int
}

// 定义了 MyType 类型的方法 SetValue，m 是接收者变量
func (m MyType) SetValue(val int) {
    m.A = val
}
```

在上面的示例中，`m` 是 `MyType` 类型的接收者变量。`SetValue` 是附加到 `MyType` 上的方法，可以通过 `MyType` 类型的变量 `m` 调用。值得注意的是，因为 `SetValue` 方法使用的是值接收者，所以这个函数内部对 `m` 的修改不会影响原始 `MyType` 变量。

没有接收者的函数，我们通常称为普通函数或者包函数。它们和方法在语法和使用场景上都有所不同。你可以理解为，普通函数更偏向于处理一般逻辑，而方法则是处理与特定类型相关的行为。



## Template