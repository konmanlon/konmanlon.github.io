# Error Handling

## error 类型

Go 语言中的错误处理是用接口实现的，源码示例：

```go
// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
	Error() string
}
```

该接口定义了一个 `Error() string` 方法，任何实现该接口的类型都可以调用此方法返回错误信息。

## 自定义错误

示例：

```go
package main

import "fmt"

type errorCustom struct {
	e string
}

// 实现 error 接口
func (s *errorCustom) Error() string {
	return s.e
}

// 构造一个错误实例，并返回一个实现了 error 接口的结构体实例的指针
func NewErr(text string) error {
	return &errorCustom{e: text}
}

func main() {
	a := NewErr("I am a error !")
	fmt.Println(a.Error())
}

```



参阅 `errors.New()`:

```go
package errors

// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```



## error 类型转换



使用[`errors.As()`](https://pkg.go.dev/errors#As)将 error 转换为原始类型，并从结构体字段中获取信息



示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Open("/test.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(f.Name(), "opened successfully")
}

```

输出结果

```tex
open /test.txt: no such file or directory
```



[`os.Open()`](https://pkg.go.dev/os#Open)函数如果有错误，则会返回一个类型为[`*PathError`](https://cs.opensource.google/go/go/+/refs/tags/go1.20.5:src/io/fs/fs.go;l=243)错误，该类型是一个结构体：

```go
// PathError records an error and the operation and file path that caused it.
type PathError struct {
	Op   string
	Path string
	Err  error
}

func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }

func (e *PathError) Unwrap() error { return e.Err }
```



使用[`errors.As()`](https://pkg.go.dev/errors#As)函数转换 error，转换成功返回 true，失败返回 false

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func main() {
	if _, err := os.Open("/test.txt"); err != nil {
        // PathError 实现了 error 接口，Error() 方法是指针接收者
		var pathErr *os.PathError
        // As 的第二个参数必须是指向实现错误的类型或接口类型的非空指针，否则会 panic
		if errors.As(err, &pathErr) {
			fmt.Println("Failed to open file at path:", pathErr.Path)
			return
		}
		fmt.Println("Generic error", err)
		return
	}

	fmt.Println("opened successfully")
}
```

输出结果

```go
Failed to open file at path: /test.txt
```

