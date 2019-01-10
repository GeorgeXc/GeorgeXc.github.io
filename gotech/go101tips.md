# Go语言的101个技巧

## 如何强迫用户使用结构的字段名称构造结构体

包开发者可用设置一个非导出的，占用0字节的字段在一个结构体中。这将会是编译器禁止包使用者不设置字段名称的情况下创建这种结构体

例子如下:

```go
// lib.go
package lib

type Config struct {
    _ [0]int
    Name string 
    Size int
}
```

```go
//main.go
package main

import (
    "lib"
)

func main(){
    //_ = lib.Config{[0]int{},"George",123} //doesn't compile
    _ = lib.Config{Name: "George",Size:123} // compile ok
}
```