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

## 不要使用有依赖关系的表达式给变量赋值

现在的go(go1.11)，在多个变量赋值时存在估值顺序，并且在标准的go编译器和gccgo编译器之间存在争议和错误。所以如果有这种情况，请把他拆分成多个值赋值给多个单一value的赋值。否则，你将无法确认是否存在依赖和设计的表达式。

事实上，一些写得差的单值赋值表达式中，也有一个表达式估值顺序分歧。例如，下面的程序可能大于[7,0,9],[0,8,9],[7,8,9],这取决于编译器的实现

```go
package main

import "fmt"

var a = &[]int{1,2,3}
var i int 

func f() int{
    i=1
    a=&[]int{7,8,9}
    return 0
}

func main(){
    //下面的a,i,f()估值顺序没有指定
    (*a)[i]=f()
    fmt.Println(*a)
}
```

换句话说，在赋值中一个函数调用不应该影响非函数调用表达式的赋值结果，详细原因请参考[赋值顺序] [https://go101.org/article/evaluation-orders.html]