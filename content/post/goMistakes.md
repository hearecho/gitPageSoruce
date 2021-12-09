---
title: "Go常见错误及处理方法"
date: 2021-12-08T19:41:14+08:00
draft: false
description: "Go常见错误的处理方法"
tags:
- Go
- 基础
---

## Go常见错误

原文链接：[50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs ](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/index.html)

### 初级

> 不熟悉Go语言可能会犯的错误

#### 1. 大括号问题

在大多数语言中，我们都可以将大括号放在任意位置，但是Go不同，Go不能将左括号放到新的一行。同时Go和Python相同是不需要分号的（即使含有分号也不会报错）。示例如下：

```Go
package main
import "fmt"
func main()  
{ //error, 不能在新的一行放置新的括号，必须紧跟函数之后
    fmt.Println("hello there!")
}
// 错误信息
// syntax error: unexpected semicolon or newline before {

// 正确语句
package main
import "fmt"
func main()  {
    fmt.Println("hello there!")
}
```

#### 2.  未使用的变量

在Go中如果出现没有被使用的变量会无法完成编译。如果在函数中声明了变量则必须使用，但是全局变量不适用则不会出现问题。如果将一个新的值分配给一个未使用的变量并不算做使用该变量。示例如下：

```go
package main

var gvar int //not an error

func main() {  
    var one int   //error, unused variable
    two := 2      //error, unused variable
    var three int //error, even though it's assigned 3 on the next line
    three = 3

    func(unused string) {
        fmt.Println("Unused arg. No compile error")
    }("what?")
}
//正常代码 想办法“使用”变量 当然如果这个变量确实一点用处都没有，则可以考虑移除
package main

import "fmt"

func main() {  
    var one int
    _ = one

    two := 2 
    fmt.Println(two)

    var three int 
    three = 3
    one = three

    var four int
    four = four
}
```

#### 3.未使用的导入的包

在使用集成开发环境的时间，一般包都是自动导自动删除的，所以这个问题一般不会出现。Go也不会允许出现未使用的包，一般情况我们都会将不适用的包删除或者是注释掉。但是在一些特殊的情况，却需要只导入包，但是并不使用他（例如连接数据库的包），所以我们一般情况下使用，`_`，作为包的名字（别名）。使用`goimports`可以直接对文件进行处理。示例如下：

```go
package main

import (  
    "fmt"
    "log"
    "time"
)

func main() {  
}
// 错误信息
// imported and not used: "fmt" 
// imported and not used: "log" 
// imported and not used: "time"
// 正确代码
package main

import (  
    _ "fmt"
    "log"
    "time"
)

var _ = log.Println

func main() {  
    _ = time.Now
}
```

#### 4. 短声明只能在函数内部使用

Go中声明方式有两种，一种是短声明，另外一种是正常声明。短声明只能在函数内部使用，所以说全局变量一般是使用正常声明方式。示例如下：

```go
package main

myvar := 1 //error

func main() {  
}
//错误信息
//non-declaration statement outside function body
// 正确声明
package main

var myvar = 1

func main() {  
}
```

#### 5. 使用短声明对变量进行了重新声明

在Go中我们不能对一个变量重新声明，即使重新声明是相同的数据类型。但在至少声明一个新变量的多变量声明中是允许的。示例如下：

```go
package main

func main() {  
    one := 0
    one := 1 //error
}
// 错误信息
// no new variables on left side of :=
// 正确声明
package main

func main() {  
    one := 0
    one, two := 1,2

    one,two = two,one
}
```

#### 6. 不能使用短声明来填充结构体中字段变量

短声明是不可以直接用于结构体中的字段变量，一般情况下我们都是使用临时变量先赋值后声明。示例如下：

```go
package main

import (  
  "fmt"
)

type info struct {  
  result int
}

func work() (int,error) {  
    return 13,nil  
  }

func main() {  
  var data info

  data.result, err := work() //error
  fmt.Printf("info: %+v\n",data)
}
// 错误信息
// non-name data.result on left side of :=
// 正确示例
package main

import (  
  "fmt"
)

type info struct {  
  result int
}

func work() (int,error) {  
    return 13,nil  
  }

func main() {  
  var data info

  var err error
  data.result, err = work() //ok
  if err != nil {
    fmt.Println(err)
    return
  }

  fmt.Printf("info: %+v\n",data) //prints: info: {result:13}
}
```

#### 7. 意外隐藏变量

短声明语法是非常方便，因此很容易将其视为常规赋值操作。如果您在新代码块中犯此错误，则不会出现编译器错误，但您的应用程序将不会按照您的预期运行。这是一个非常普通的陷阱，它很容易出错但是不容易被发现，您可以使用 vet 命令来查找其中一些问题。默认情况下，vet 不会执行任何隐藏变量检查。确保使用 -shadow 标志：go tool vet -shadow your_file.go。请注意， vet 命令不会报告所有隐藏的变量。使用 go-nyet 进行更积极的隐藏变量检测。示例如下：

```go
package main

import "fmt"

func main() {  
    x := 1
    fmt.Println(x)     //prints 1
    // 代码块所以 第一条错误在这里不会出现
    {
        fmt.Println(x) //prints 1
        x := 2
        fmt.Println(x) //prints 2
    }
    fmt.Println(x)     //prints 1 (bad if you need 2)
}

```

#### 8. 不能使用nil来初始化没有显式类型的变量

例如接口、函数、指针、哈希表、切片和通道的默认值是`nil`，但是如果我们没有指明一个变量的类型却将`nil`赋值或用于其初始化则是行不通的，因为Go无法推断出他的类型。示例如下：

```go
package main

func main() {  
    var x = nil //error

    _ = x
}
// 错误信息
// use of untyped nil
// 正确示例
package main

func main() {  
    var x interface{} = nil

    _ = x
}
```

#### 9. 使用“nil”的切片和哈希表

直接向"nil"切片中添加元素是没有问题的，但是向"nil"的哈希表中添加元素会产生运行时错误。示例如下：

```go
package main

func main() {  
    var m map[string]int
    m["one"] = 1 //error

}
// slice
package main

func main() {  
    var s []int
    s = append(s,1)
}
```

#### 10. 哈希表容量

我们在创建哈希表的时候可以指定哈希表的容量，但是函数`cap()`无法在哈希表上使用。示例如下：

```go
package main

func main() {  
    m := make(map[string]int,99)
    cap(m) //error
}
// 错误信息
// invalid argument m (type map[string]int) for cap
// cap接收的参数是 Array、Pointer、Slice、 Channel
```



