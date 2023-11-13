# map



示例：获取嵌套 map 的数据

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	data := []byte(`{"hello":"world","person":{"name":"vincent","age":20}}`)
	// fmt.Printf("%s\n", data)
	var s map[string]any
	err := json.Unmarshal(data, &s)
	if err == nil {
		fmt.Printf("%v\n", s)
		//t := s["person"].(map[string]interface{})
		fmt.Printf("%v\n", s["person"].(map[string]any)["age"])
        // 将 any 类型的 n 转换为 string，如果转换失败程序会异常退出
        n := s["person"].(map[string]any)["name"].(string)
        fmt.Printf("%T", n)
	} else {
		fmt.Println(err)
	}
}

```

