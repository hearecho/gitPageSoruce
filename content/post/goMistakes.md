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

#### 11. 字符串不可以被初始化赋值为nil

字符串的默认空值是`""`，而不是`nil`。示例如下：

```go
func main() {  
    var x string = nil //error

    if x == nil { //error
        x = "default"
    }
}
// 错误信息
// cannot use nil as type string in assignment
// invalid operation: x == nil (mismatched types string and nil)
// 正确做法
func main() {  
    var x string //defaults to "" (zero value)

    if x == "" {
        x = "default"
    }
}
```

#### 12. 数组参数

数组作为函数参数的时候是值复制，所以在函数内部修改数组的值是不会产生同步的修改。如果想要达到在函数内部的修改可以同步，则可以使用指针或者是使用切片。示例如下：

```go
func main() {  
    x := [3]int{1,2,3}

    func(arr [3]int) {
        arr[0] = 7
        fmt.Println(arr) //prints [7 2 3]
    }(x)

    fmt.Println(x) //prints [1 2 3] (not ok if you need [7 2 3])
}
// 正确用法
// 使用指针
func main() {  
    x := [3]int{1,2,3}

    func(arr *[3]int) {
        (*arr)[0] = 7
        fmt.Println(arr) //prints &[7 2 3]
    }(&x)

    fmt.Println(x) //prints [7 2 3]
}
// 使用切片 实际上切片也是传入的指针参数
func main() {  
    x := []int{1,2,3}

    func(arr []int) {
        arr[0] = 7
        fmt.Println(arr) //prints [7 2 3]
    }(x)

    fmt.Println(x) //prints [7 2 3]
}
```

#### 13.  使用`range`遍历数组和切片时候的意外值

使用`range`遍历数组和切片的时候是返回索引和该索引对应的值一组键值对。第二位才是我们需要的值。第一位是索引。示例如下：

```go
func main() {  
    x := []string{"a","b","c"}

    for v := range x {
        fmt.Println(v) //prints 0, 1, 2
    }
}
// 正确用法
func main() {  
    x := []string{"a","b","c"}

    for _, v := range x {
        fmt.Println(v) //prints a, b, c
    }
}
```

#### 14. 数组和切片都是一维的

看起来Go支持多维数组和切片，但它并不支持。不过创建数组的数组或切片的切片是可能的。对于依赖动态多维数组的数值计算应用来说，在性能和复杂性方面都远非理想。你可以使用原始一维数组、"独立 "切片和 "共享数据 "切片来构建动态多维数组。如果你使用的是原始的一维数组，你要负责索引、边界检查，以及当数组需要增长时的内存重新分配。使用 "独立 "切片创建一个动态多维数组是一个两步过程。首先，你必须创建外层片。然后，你必须分配每个内片。内片是相互独立的。你可以在不影响其他内片的情况下增长和缩小它们。示例如下：

```go
func main() {  
    x := 2
    y := 4

    table := make([][]int,x)
    for i:= range table {
        table[i] = make([]int,y)
    }
}
// 但是对于其他语言来说，底层是数据连续的层次
// 证明
func main() {  
    h, w := 2, 4
    raw := make([]int, h*w)
	for i := range raw {
		raw[i] = i
	}
	fmt.Println(raw, &raw[3], &raw[4])
	//prints: [0 1 2 3 4 5 6 7]  0xc000010318 0xc000010320

	table := make([][]int, h)
	for i := range table {
		table[i] = raw[i*w : i*w+w]
	}
	fmt.Println(table, &table[0][3], &table[1][0])
    // prints [[0 1 2 3] [4 5 6 7]] 0xc000010318 0xc000010320
}
```

#### 15. 访问不存在的哈希表键

一般情况下我们期望访问不存在的哈希表键的时候期望能够返回值为`nil`，但是返回`nil`的情况是那些value值的默认值为`nil`，但是还有很多数据类型的默认值不为`nil`。所以正确的做法是先辨别哈希表中是否存在哈希表键，之后再进行处理。示例如下：

```go
// 错误示范
func main() {  
    x := map[string]string{"one":"a","two":"","three":"c"}

    if v := x["two"]; v == "" { //incorrect
        fmt.Println("no entry")
    }
}
// 正确做法
func main() {  
    x := map[string]string{"one":"a","two":"","three":"c"}

    if _,ok := x["two"]; !ok {
        fmt.Println("no entry")
    }
}
```

#### 16. 字符串是不可变类型

尝试直接更新字符串中某个字符是不可行的，字符串作为不可变类型是只能读取但是不能修改。如果需要更新一个字符串，则可以选择先将其转换为字节切片，之后在需要的时候转换为字符串。示例如下：

```go
//错误示范
func main() {  
    x := "text"
    x[0] = 'T'

    fmt.Println(x)
}
//错误信息
//annot assign to x[0]
//正确示范
func main() {  
    x := "text"
    xbytes := []byte(x)
    xbytes[0] = 'T'

    fmt.Println(string(xbytes)) //prints Text
}

```

*注意：*这对于文本字符串来说并不是一个好的更新方式，因为文本字符串会存储在多个字节中。如果确实需要更新文本字符串，可以先将其转换为`rune`切片，但是即使使用`rune`切片也有可能会出现占据多个`rune`的情况。

#### 17.  字符串和字节切片之间的转换

当你将一个字符串转换为一个字节片时（反之亦然），你会得到一个原始数据的完整拷贝。这不像其他语言中的转换操作，也不像重新切分那样，新的切分变量指向原始字节切分所使用的同一个底层数组。

Go确实对`[]byte`到`string`和`string`到`[]byte`的转换进行了一些优化，以避免额外的分配（在todo列表中还有更多优化）。

第一个优化避免了在`map[string]`集合中使用`[]byte`键来查找条目时的额外分配：`m[string(key)]`。

第二个优化避免了`string`被转换为`[]byte`的`for range`子句中的额外分配：`for i,v := range []byte(str) {...}`.

#### 18. 字符串与索引操作

直接使用索引操作得到是一个`byte`值而不是字符。示例如下：

```go
func main() {  
    x := "text"
    fmt.Println(x[0]) //print 116
    fmt.Printf("%T",x[0]) //prints uint8
}
```

如果想要得到字符，则可以使用`for range`短语，官方的 "unicode/utf8 "包和实验性的utf8string包（golang.org/x/exp/utf8string）也很有用。utf8string包包括一个方便的`At()`方法。将字符串转换为符文片也是一种选择。

#### 19. 字符串不一定都是UTF8文本

字符串值不要求是UTF8文本。它们可以包含任意的字节。只有在使用字符串字面的时候，字符串才是UTF8的。即使如此，它们也可以使用转义序列包含其他数据。要知道你是否有一个UTF8文本字符串，请使用 "unicode/utf8 "包中的ValidString()函数。 示例如下：

```go
func main() {  
    data1 := "ABC"
    fmt.Println(utf8.ValidString(data1)) //prints: true

    data2 := "A\xfeC"
    fmt.Println(utf8.ValidString(data2)) //prints: false
}
```

#### 20. 字符串长度

Go内置的`len()`函数返回值字节的数量而不是python中的字符的数量。为了得到字符的数量我们一般使用"unicode/utf8"包中的`RuneCountInString()`函数。示例如下：

```go
func main() {  
    data := "♥"
    fmt.Println(utf8.RuneCountInString(data))
}
```

*注意：*这个和上面转换一样存在意外情况就是字符串中含有`é`等类似字符，因为其占用两个字符。



