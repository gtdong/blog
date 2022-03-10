---
title:  Go-基础
tags:
  - Go
categories:
  - Go语言
date: 2021-05-18 13:05:00
---


# 1. 介绍与安装

## Golang 是什么

Go 亦称为 Golang（按照 Rob Pike 说法，语言叫做 Go，Golang 只是官方网站的网址），是由谷歌开发的一个开源的编译型的静态语言。

Golang 的主要关注点是使得高可用性和可扩展性的 Web 应用的开发变得简便容易。（Go 的定位是系统编程语言，只是对 Web 开发支持较好）

<!--more-->

## 为何选择 Golang

既然有很多其他编程语言可以做同样的工作，如 Python，Ruby，Nodejs 等，为什么要选择 Golang 作为服务端编程语言？

以下是我使用 Go 语言时发现的一些优点：

- 并发是语言的一部分（并非通过标准库实现），所以编写多线程程序会是一件很容易的事。后续教程将会讨论到，并发是通过 Goroutines 和 channels 机制实现的。
- Golang 是一种编译型语言。源代码会编译为二进制机器码。而在解释型语言中没有这个过程，如 Nodejs 中的 JavaScript。
- 语言规范十分简洁。所有规范都在一个页面展示，你甚至都可以用它来编写你自己的编译器呢。
- Go 编译器支持静态链接。所有 Go 代码都可以静态链接为一个大的二进制文件（相对现在的磁盘空间，其实根本不大），并可以轻松部署到云服务器，而不必担心各种依赖性。

## 安装

Golang 支持三个平台：Mac，Windows 和 Linux（译注：不只是这三个，也支持其他主流平台）。你可以在 https://golang.org/dl/ 中下载相应平台的二进制文件。（因为众所周知的原因，如果下载不了，请到 https://studygolang.com/dl 下载）

### Mac OS

在 https://golang.org/dl/ 下载安装程序。双击开始安装并且遵循安装提示，会将 Golang 安装到 `/usr/local/go` 目录下，同时 `/usr/local/go/bin` 文件夹也会被添加到 `PATH` 环境变量中。

### Windows

在 https://golang.org/dl/ 下载 MSI 安装程序。双击开始安装并且遵循安装提示，会将 Golang 安装到 `C:\Go` 目录下，同时 `c:\Go\bin` 目录也会被添加到你的 `PATH` 环境变量中。

### Linux

在 https://golang.org/dl/ 下载 tar 文件，并解压到 `/usr/local`。

请添加 `/usr/local/go/bin` 到 `PATH` 环境变量中。Go 就已经成功安装在 `Linux` 上了。

# 2. Hello World

## 建立 Go 工作区

在编写代码之前，我们首先应该建立 Go 的工作区（Workspace）。

在 **Mac 或 Linux** 操作系统下，Go 工作区应该设置在 **𝐻𝑂𝑀𝐸/𝑔𝑜∗∗。所以我们要在∗∗HOME/go∗∗。所以我们要在∗∗HOME** 目录下创建 **go** 目录。

而在 **Windows** 下，工作区应该设置在 **C:\Users\YourName\go**。所以请将 **go** 目录放置在 **C:\Users\YourName**。

其实也可以通过设置 GOPATH 环境变量，用其他目录来作为工作区。但为了简单起见，我们采用上面提到的放置方法。

所有 Go 源文件都应该放置在工作区里的 **src** 目录下。请在刚添加的 **go** 目录下面创建目录 **src**。

所有 Go 项目都应该依次在 src 里面设置自己的子目录。我们在 src 里面创建一个目录 **hello** 来放置整个 hello world 项目。

创建上述目录之后，其目录结构如下：

```
Copygo
  src
    hello
```

在我们刚刚创建的 hello 目录下，在 **helloworld.go** 文件里保存下面的程序。

```go
Copypackage main

import "fmt"

func main() {  
    fmt.Println("Hello World")
}
```

创建该程序之后，其目录结构如下：

```
Copygo
  src
    hello
      helloworld.go
```

## 运行 Go 程序

运行 Go 程序有多种方式，我们下面依次介绍。

1.使用 **go run** 命令 - 在命令提示符旁，输入 `go run workspacepath/src/hello/helloworld.go`。

上述命令中的 **workspacepath** 应该替换为你自己的工作区路径（Windows 下的 **C:/Users/YourName/go**，Linux 或 Mac 下的 **$HOME/go**）。

在控制台上会看见 `Hello World` 的输出。

2.使用 **go install** 命令 - 运行 `go install hello`，接着可以用 `workspacepath/bin/hello` 来运行该程序。

上述命令中的 **workspacepath** 应该替换为你自己的工作区路径（Windows 下的 **C:/Users/YourName/go**，Linux 或 Mac 下的 **$HOME/go**）。

当你输入 **go install hello** 时，go 工具会在工作区中搜索 hello 包（hello 称之为包，我们后面会更加详细地讨论包）。接下来它会在工作区的 bin 目录下，创建一个名为 `hello`（Windows 下名为 `hello.exe`）的二进制文件。运行 **go install hello** 后，其目录结构如下所示：

```
Copygo
  bin
    hello
  src
    hello
      helloworld.go
```

3.第 3 种运行程序的好方法是使用 go playground。尽管它有自身的限制，但该方法对于运行简单的程序非常方便。我已经在 playground 上创建了一个 hello world 程序。[点击这里](https://play.golang.org/p/VtXafkQHYe) 在线运行程序。 你可以使用 [go playground](https://play.golang.org/) 与其他人分享你的源代码。

### 简述 hello world 程序

下面就是我们刚写下的 hello world 程序。

```go
Copypackage main //1

import "fmt" //2

func main() { //3  
    fmt.Println("Hello World") //4
}
```

现在简单介绍每一行大概都做了些什么，在以后的教程中还会深入探讨每个部分。

**package main - 每一个 Go 文件都应该在开头进行 package name 的声明**（译注：只有可执行程序的包名应当为 main）。包（Packages）用于代码的封装与重用，这里的包名称是`main`。

**import "fmt"** - 我们引入了 fmt 包，用于在 main 函数里面打印文本到标准输出。

**func main()** - main 是一个特殊的函数。整个程序就是从 main 函数开始运行的。**main 函数必须放置在 main 包中**。`{` 和 `}` 分别表示 main 函数的开始和结束部分。

**fmt.Println("Hello World")** - **fmt** 包中的 **Println** 函数用于把文本写入标准输出。

# 3. 变量

### 变量是什么

变量指定了某存储单元（Memory Location）的名称，该存储单元会存储特定类型的值。在 Go 中，有多种语法用于声明变量。

### 声明单个变量

**var name type** 是声明单个变量的语法。

```go
Copypackage main

import "fmt"

func main() {
    var age int // 变量声明
    fmt.Println("my age is", age)
}
```

语句 `var age int` 声明了一个 int 类型的变量，名字为 age。我们还没有给该变量赋值。如果变量未被赋值，Go 会自动地将其初始化，赋值该变量类型的零值（Zero Value）。本例中 age 就被赋值为 0。如果你运行该程序，你会看到如下输出：

```
Copymy age is 0
```

变量可以赋值为本类型的任何值。上一程序中的 age 可以赋值为任何整型值（Integer Value）。

```go
Copypackage main

import "fmt"

func main() {
    var age int // 变量声明
    fmt.Println("my age is", age)
    age = 29 // 赋值
    fmt.Println("my age is", age)
    age = 54 // 赋值
    fmt.Println("my new age is", age)
}
```

上面的程序会有如下输出：

```
Copymy age is  0  
my age is 29  
my new age is 54
```

### 声明变量并初始化

声明变量的同时可以给定初始值。 **var name type = initialvalue** 的语法用于声明变量并初始化。

```go
Copypackage main

import "fmt"

func main() {
    var age int = 29 // 声明变量并初始化

    fmt.Println("my age is", age)
}
```

在上面的程序中，age 是具有初始值 29 的 int 类型变量。如果你运行上面的程序，你可以看见下面的输出，证实 age 已经被初始化为 29。

```
Copymy age is 29
```

### 类型推断（Type Inference）

如果变量有初始值，那么 Go 能够自动推断具有初始值的变量的类型。因此，如果变量有初始值，就可以在变量声明中省略 `type`。

如果变量声明的语法是 **var name = initialvalue**，Go 能够根据初始值自动推断变量的类型。

在下面的例子中，你可以看到在第 6 行，我们省略了变量 `age` 的 `int` 类型，Go 依然推断出了它是 int 类型。

```go
Copypackage main

import "fmt"

func main() {
    var age = 29 // 可以推断类型

    fmt.Println("my age is", age)
}
```

### 声明多个变量

Go 能够通过一条语句声明多个变量。

声明多个变量的语法是 **var name1, name2 type = initialvalue1, initialvalue2**。

```go
Copypackage main

import "fmt"

func main() {
    var width, height int = 100, 50 // 声明多个变量

    fmt.Println("width is", width, "height is", heigh)
}
```

上述程序将在标准输出打印 `width is 100 height is 50`。

你可能已经想到，如果 width 和 height 省略了初始化，它们的初始值将赋值为 0。

```go
Copypackage mainimport "fmt"func main() {      var width, height int    fmt.Println("width is", width, "height is", height)    width = 100    height = 50    fmt.Println("new width is", width, "new height is ", height)}
```

上面的程序将会打印：

```
Copywidth is 0 height is 0  new width is 100 new height is  50
```

在有些情况下，我们可能会想要在一个语句中声明不同类型的变量。其语法如下：

```go
Copyvar (      name1 = initialvalue1,    name2 = initialvalue2)
```

使用上述语法，下面的程序声明不同类型的变量。

```go
Copypackage mainimport "fmt"func main() {    var (        name   = "naveen"        age    = 29        height int    )    fmt.Println("my name is", name, ", age is", age, "and height is", height)}
```

这里我们声明了 **string 类型的 name、int 类型的 age 和 height**（我们将会在下一教程中讨论 golang 所支持的变量类型）。运行上面的程序会产生输出 `my name is naveen , age is 29 and height is 0`。

### 简短声明

Go 也支持一种声明变量的简洁形式，称为简短声明（Short Hand Declaration），该声明使用了 **:=** 操作符。

声明变量的简短语法是 **name := initialvalue**。

```go
Copypackage mainimport "fmt"func main() {      name, age := "naveen", 29 // 简短声明    fmt.Println("my name is", name, "age is", age)}
```

运行上面的程序，可以看到输出为 `my name is naveen age is 29`。

简短声明要求 **:=** 操作符左边的所有变量都有初始值。下面程序将会抛出错误 `cannot assign 1 values to 2 variables`，这是因为 **age 没有被赋值**。

```go
Copypackage mainimport "fmt"func main() {      name, age := "naveen" //error    fmt.Println("my name is", name, "age is", age)}
```

简短声明的语法要求 **:=** 操作符的左边至少有一个变量是尚未声明的。考虑下面的程序：

```go
Copypackage mainimport "fmt"func main() {    a, b := 20, 30 // 声明变量a和b    fmt.Println("a is", a, "b is", b)    b, c := 40, 50 // b已经声明，但c尚未声明    fmt.Println("b is", b, "c is", c)    b, c = 80, 90 // 给已经声明的变量b和c赋新值    fmt.Println("changed b is", b, "c is", c)}
```

在上面程序中的第 8 行，由于 b 已经被声明，而 c 尚未声明，因此运行成功并且输出：

```
Copya is 20 b is 30  b is 40 c is 50  changed b is 80 c is 90
```

但是如果我们运行下面的程序:

```go
Copypackage mainimport "fmt"func main() {      a, b := 20, 30 // 声明a和b    fmt.Println("a is", a, "b is", b)    a, b := 40, 50 // 错误，没有尚未声明的变量}
```

上面运行后会抛出 `no new variables on left side of :=` 的错误，这是因为 a 和 b 的变量已经声明过了，**:=** 的左边并没有尚未声明的变量。

变量也可以在运行时进行赋值。考虑下面的程序：

```go
Copypackage mainimport (      "fmt"    "math")func main() {      a, b := 145.8, 543.8    c := math.Min(a, b)    fmt.Println("minimum value is ", c)}
```

在上面的程序中，c 的值是运行过程中计算得到的，即 a 和 b 的最小值。上述程序会打印：

```
Copyminimum value is  145.8
```

由于 Go 是强类型（Strongly Typed）语言，因此不允许某一类型的变量赋值为其他类型的值。下面的程序会抛出错误 `cannot use "naveen" (type string) as type int in assignment`，这是因为 age 本来声明为 int 类型，而我们却尝试给它赋字符串类型的值。

```go
Copypackage mainfunc main() {      age := 29      // age是int类型    age = "naveen" // 错误，尝试赋值一个字符串给int类型变量}
```

# 4. 类型

下面是 Go 支持的基本类型：

- bool
- 数字类型
  - int8, int16, int32, int64, int
  - uint8, uint16, uint32, uint64, uint
  - float32, float64
  - complex64, complex128
  - byte
  - rune
- string

### bool

bool 类型表示一个布尔值，值为 true 或者 false。

```go
Copypackage main

import "fmt"

func main() {  
    a := true
    b := false
    fmt.Println("a:", a, "b:", b)
    c := a && b
    fmt.Println("c:", c)
    d := a || b
    fmt.Println("d:", d)
}
```

在上面的程序中，a 赋值为 true，b 赋值为 false。

c 赋值为 a && b。仅当 a 和 b 都为 true 时，操作符 && 才返回 true。因此，在这里 c 为 false。

当 a 或者 b 为 true 时，操作符 || 返回 true。在这里，由于 a 为 true，因此 d 也为 true。我们将得到程序的输出如下。

```
Copya: true b: false  
c: false  
d: true
```

### 有符号整型

**int8**：表示 8 位有符号整型
**大小**：8 位
**范围**：-128～127

**int16**：表示 16 位有符号整型
**大小**：16 位
**范围**：-32768～32767

**int32**：表示 32 位有符号整型
**大小**：32 位
**范围**：-2147483648～2147483647

**int64**：表示 64 位有符号整型
**大小**：64 位
**范围**：-9223372036854775808～9223372036854775807

**int**：根据不同的底层平台（Underlying Platform），表示 32 或 64 位整型。除非对整型的大小有特定的需求，否则你通常应该使用 *int* 表示整型。
**大小**：在 32 位系统下是 32 位，而在 64 位系统下是 64 位。
**范围**：在 32 位系统下是 -2147483648～2147483647，而在 64 位系统是 -9223372036854775808～9223372036854775807。

```go
Copypackage main

import "fmt"

func main() {  
    var a int = 89
    b := 95
    fmt.Println("value of a is", a, "and b is", b)
}
```

[在线运行程序](https://play.golang.org/p/NyDPsjkma3)

上面程序会输出 `value of a is 89 and b is 95`。

在上述程序中，a 是 int 类型，而 b 的类型通过赋值（95）推断得出。上面我们提到，int 类型的大小在 32 位系统下是 32 位，而在 64 位系统下是 64 位。接下来我们会证实这种说法。

在 Printf 方法中，使用 **%T** 格式说明符（Format Specifier），可以打印出变量的类型。Go 的 [unsafe](https://golang.org/pkg/unsafe/) 包提供了一个 [Sizeof](https://golang.org/pkg/unsafe/#Sizeof) 函数，该函数接收变量并返回它的字节大小。*unsafe* 包应该小心使用，因为使用 unsafe 包可能会带来可移植性问题。不过出于本教程的目的，我们是可以使用的。

下面程序会输出变量 a 和 b 的类型和大小。格式说明符 `%T` 用于打印类型，而 `%d` 用于打印字节大小。

```go
Copypackage main

import (  
    "fmt"
    "unsafe"
)

func main() {  
    var a int = 89
    b := 95
    fmt.Println("value of a is", a, "and b is", b)
    fmt.Printf("type of a is %T, size of a is %d", a, unsafe.Sizeof(a)) // a 的类型和大小
    fmt.Printf("\ntype of b is %T, size of b is %d", b, unsafe.Sizeof(b)) // b 的类型和大小
}
```

[在线运行程序](https://play.golang.org/p/mFsmjVk5oc)

以上程序会输出：

```
Copyvalue of a is 89 and b is 95  
type of a is int, size of a is 4  
type of b is int, size of b is 4
```

从上面的输出，我们可以推断出 a 和 b 为 *int* 类型，且大小都是 32 位（4 字节）。如果你在 64 位系统上运行上面的代码，会有不同的输出。在 64 位系统下，a 和 b 会占用 64 位（8 字节）的大小。

### 无符号整型

**uint8**：表示 8 位无符号整型
**大小**：8 位
**范围**：0～255

**uint16**：表示 16 位无符号整型
**大小**：16 位
**范围**：0～65535

**uint32**：表示 32 位无符号整型
**大小**：32 位
**范围**：0～4294967295

**uint64**：表示 64 位无符号整型
**大小**：64 位
**范围**：0～18446744073709551615

**uint**：根据不同的底层平台，表示 32 或 64 位无符号整型。
**大小**：在 32 位系统下是 32 位，而在 64 位系统下是 64 位。
**范围**：在 32 位系统下是 0～4294967295，而在 64 位系统是 0～18446744073709551615。

### 浮点型

**float32**：32 位浮点数
**float64**：64 位浮点数

下面一个简单程序演示了整型和浮点型的运用。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    a, b := 5.67, 8.97
    fmt.Printf("type of a %T b %T\n", a, b)
    sum := a + b
    diff := a - b
    fmt.Println("sum", sum, "diff", diff)

    no1, no2 := 56, 89
    fmt.Println("sum", no1+no2, "diff", no1-no2)
}
```

a 和 b 的类型根据赋值推断得出。在这里，a 和 b 的类型为 float64（float64 是浮点数的默认类型）。我们把 a 和 b 的和赋值给变量 sum，把 b 和 a 的差赋值给 diff，接下来打印 sum 和 diff。no1 和 no2 也进行了相同的计算。上述程序将会输出：

```
Copytype of a float64 b float64  
sum 14.64 diff -3.3000000000000007  
sum 145 diff -33
```

### 复数类型

**complex64**：实部和虚部都是 float32 类型的的复数。
**complex128**：实部和虚部都是 float64 类型的的复数。

内建函数 **complex**用于创建一个包含实部和虚部的复数。complex 函数的定义如下：

```
Copyfunc complex(r, i FloatType) ComplexType
```

该函数的参数分别是实部和虚部，并返回一个复数类型。实部和虚部应该是相同类型，也就是 float32 或 float64。如果实部和虚部都是 float32 类型，则函数会返回一个 complex64 类型的复数。如果实部和虚部都是 float64 类型，则函数会返回一个 complex128 类型的复数。

还可以使用简短语法来创建复数：

```
Copyc := 6 + 7i
```

下面我们编写一个简单的程序来理解复数。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    c1 := complex(5, 7)
    c2 := 8 + 27i
    cadd := c1 + c2
    fmt.Println("sum:", cadd)
    cmul := c1 * c2
    fmt.Println("product:", cmul)
}
```

在上面的程序里，c1 和 c2 是两个复数。c1的实部为 5，虚部为 7。c2 的实部为8，虚部为 27。c1 和 c2 的和赋值给 `cadd` ，而 c1 和 c2 的乘积赋值给 `cmul`。该程序将输出：

```
Copysum: (13+34i)  
product: (-149+191i)
```

### 其他数字类型

**byte** 是 uint8 的别名。
**rune** 是 int32 的别名。

在学习字符串的时候，我们会详细讨论 byte 和 rune。

### string 类型

在 Golang 中，字符串是字节的集合。如果你现在还不理解这个定义，也没有关系。我们可以暂且认为一个字符串就是由很多字符组成的。我们后面会在一个教程中深入学习字符串。 下面编写一个使用字符串的程序。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    first := "Naveen"
    last := "Ramanathan"
    name := first +" "+ last
    fmt.Println("My name is",name)
}
```

上面程序中，first 赋值为字符串 "Naveen"，last 赋值为字符串 "Ramanathan"。+ 操作符可以用于拼接字符串。我们拼接了 first、空格和 last，并将其赋值给 name。上述程序将打印输出 `My name is Naveen Ramanathan`。

还有许多应用于字符串上面的操作，我们将会在一个单独的教程里看见它们。

### 类型转换

Go 有着非常严格的强类型特征。Go 没有自动类型提升或类型转换。我们通过一个例子说明这意味着什么。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    i := 55      //int
    j := 67.8    //float64
    sum := i + j //不允许 int + float64
    fmt.Println(sum)
}
```

上面的代码在 C 语言中是完全合法的，然而在 Go 中，却是行不通的。i 的类型是 int ，而 j 的类型是 float64 ，我们正试图把两个不同类型的数相加，Go 不允许这样的操作。如果运行程序，你会得到 `main.go:10: invalid operation: i + j (mismatched types int and float64)`。

要修复这个错误，i 和 j 应该是相同的类型。在这里，我们把 j 转换为 int 类型。把 v 转换为 T 类型的语法是 T(v)。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    i := 55      //int
    j := 67.8    //float64
    sum := i + int(j) //j is converted to int
    fmt.Println(sum)
}
```

现在，当你运行上面的程序时，会看见输出 `122`。

赋值的情况也是如此。把一个变量赋值给另一个不同类型的变量，需要显式的类型转换。下面程序说明了这一点。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    i := 10
    var j float64 = float64(i) // 若没有显式转换，该语句会报错
    fmt.Println("j", j)
}
```

在第 9 行，i 转换为 float64 类型，接下来赋值给 j。如果不进行类型转换，当你试图把 i 赋值给 j 时，编译器会抛出错误。

# 5. 常量

### 定义

在 Go 语言中，术语"常量"用于表示固定的值。比如 `5` 、`-89`、 `I love Go`、`67.89` 等等。

看看下面的代码:

```go
Copyvar a int = 50  
var b string = "I love Go"
```

**在上面的代码中，变量 a 和 b 分别被赋值为常量 50 和 I love GO**。关键字 `const` 被用于表示常量，比如 `50` 和 `I love Go`。即使在上面的代码中我们没有明确的使用关键字 `const`，但是在 Go 的内部，它们是常量。

顾名思义，常量不能再重新赋值为其他的值。因此下面的程序将不能正常工作，它将出现一个编译错误: `cannot assign to a.`。

```go
Copypackage main

func main() {  
    const a = 55 // 允许
    a = 89       // 不允许重新赋值
}
```

常量的值会在编译的时候确定。因为函数调用发生在运行时，所以不能将函数的返回值赋值给常量。

```go
Copypackage main

import (  
    "fmt"
    "math"
)

func main() {  
    fmt.Println("Hello, playground")
    var a = math.Sqrt(4)   // 允许
    const b = math.Sqrt(4) // 不允许
}
```

在上面的程序中，因为 `a` 是变量，因此我们可以将函数 `math.Sqrt(4)` 的返回值赋值给它（我们将在单独的地方详细讨论函数）。

```
b` 是一个常量，它的值需要在编译的时候就确定。函数 `math.Sqrt(4)` 只会在运行的时候计算，因此 `const b = math.Sqrt(4)` 将会抛出错误 `error main.go:11: const initializer math.Sqrt(4) is not a constant)
```

### 字符串常量

双引号中的任何值都是 Go 中的字符串常量。例如像 `Hello World` 或 `Sam` 等字符串在 Go 中都是常量。

什么类型的字符串属于常量？答案是他们是无类型的。

像 `Hello World` 这样的字符串常量没有任何类型。

```go
Copyconst hello = "Hello World"
```

上面的例子，我们把 `Hello World` 分配给常量 `hello`。现在常量 `hello` 有类型吗？答案是没有。常量仍然没有类型。

Go 是一门强类型语言，所有的变量必须有明确的类型。那么, 下面的程序是如何将无类型的常量 `Sam` 赋值给变量 `name` 的呢？

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    var name = "Sam"
    fmt.Printf("type %T value %v", name, name)

}
```

**答案是无类型的常量有一个与它们相关联的默认类型，并且当且仅当一行代码需要时才提供它。在声明中 var name = "Sam" ， name 需要一个类型，它从字符串常量 Sam 的默认类型中获取。**

有没有办法创建一个带类型的常量？答案是可以的。以下代码创建一个有类型常量。

```go
Copyconst typedhello string = "Hello World"
```

上面代码中， `typedhello` 就是一个 `string` 类型的常量。

Go 是一个强类型的语言，在分配过程中混合类型是不允许的。让我们通过以下程序看看这句话是什么意思。

```go
Copypackage main

func main() {  
        var defaultName = "Sam" // 允许
        type myString string
        var customName myString = "Sam" // 允许
        customName = defaultName // 不允许

}
```

在上面的代码中，我们首先创建一个变量 `defaultName` 并分配一个常量 `Sam` 。**常量 Sam 的默认类型是 string ，所以在赋值后 defaultName 是 string 类型的。**

下一行，我们将创建一个新类型 `myString`，它是 `string` 的别名。

然后我们创建一个 `myString` 的变量 `customName` 并且给他赋值一个常量 `Sam` 。因为常量 `Sam` 是无类型的，它可以分配给任何字符串变量。因此这个赋值是允许的，`customName` 的类型是 `myString`。

现在，我们有一个类型为 `string` 的变量 `defaultName` 和另一个类型为 `myString` 的变量 `customName`。即使我们知道这个 `myString` 是 `string` 类型的别名。Go 的类型策略不允许将一种类型的变量赋值给另一种类型的变量。因此将 `defaultName` 赋值给 `customName` 是不允许的，编译器会抛出一个错误 `main.go:7:20: cannot use defaultName (type string) as type myString in assignmen`。

### 布尔常量

布尔常量和字符串常量没有什么不同。他们是两个无类型的常量 `true` 和 `false`。字符串常量的规则适用于布尔常量，所以在这里我们不再重复。以下是解释布尔常量的简单程序。

```go
Copypackage mainfunc main() {      const trueConst = true    type myBool bool    var defaultBool = trueConst // 允许    var customBool myBool = trueConst // 允许    defaultBool = customBool // 不允许}
```

上面的程序是自我解释的。

### 数字常量

数字常量包含整数、浮点数和复数的常量。数字常量中有一些微妙之处。

让我们看一些例子来说清楚。

```go
Copypackage mainimport (      "fmt")func main() {      const a = 5    var intVar int = a    var int32Var int32 = a    var float64Var float64 = a    var complex64Var complex64 = a    fmt.Println("intVar",intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var",complex64Var)}
```

上面的程序，常量 `a` 是没有类型的，它的值是 `5` 。您可能想知道 `a` 的默认类型是什么，如果它确实有一个的话, 那么我们如何将它分配给不同类型的变量。答案在于 `a` 的语法。下面的程序将使事情更加清晰。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    var i = 5
    var f = 5.6
    var c = 5 + 6i
    fmt.Printf("i's type %T, f's type %T, c's type %T", i, f, c)

}
```

在上面的程序中，每个变量的类型由数字常量的语法决定。`5` 在语法中是整数， `5.6` 是浮点数，`5+6i` 的语法是复数。当我们运行上面的程序，它会打印出 `i's type int, f's type float64, c's type complex128`。

现在我希望下面的程序能够正确的工作。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    const a = 5
    var intVar int = a
    var int32Var int32 = a
    var float64Var float64 = a
    var complex64Var complex64 = a
    fmt.Println("intVar",intVar, "\nint32Var", int32Var, "\nfloat64Var", float64Var, "\ncomplex64Var",complex64Var)
}
```

在这个程序中， `a` 的值是 `5` ，`a` 的语法是通用的（它可以代表一个浮点数、整数甚至是一个没有虚部的复数），因此可以将其分配给任何兼容的类型。这些常量的默认类型可以被认为是根据上下文在运行中生成的。 `var intVar int = a` 要求 `a` 是 `int`，所以它变成一个 `int` 常量。 `var complex64Var complex64 = a` 要求 `a` 是 `complex64`，因此它变成一个复数类型。很简单的:)。

### 数字表达式

数字常量可以在表达式中自由混合和匹配，只有当它们被分配给变量或者在需要类型的代码中的任何地方使用时，才需要类型。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    var a = 5.9/8
    fmt.Printf("a's type %T value %v",a, a)
}
```

在上面的程序中， `5.9` 在语法中是浮点型，`8` 是整型，`5.9/8` 是允许的，因为两个都是数字常量。除法的结果是 `0.7375` 是一个浮点型，所以 `a` 的类型是浮点型。这个程序的输出结果是: `a's type float64 value 0.7375`。

# 6. 函数（Function）

## 函数是什么？

函数是一块执行特定任务的代码。一个函数是在输入源基础上，通过执行一系列的算法，生成预期的输出。

## 函数的声明

在 Go 语言中，函数声明通用语法如下：

```go
Copyfunc functionname(parametername type) returntype {  
    // 函数体（具体实现的功能）
}
```

函数的声明以关键词 `func` 开始，后面紧跟自定义的函数名 `functionname (函数名)`。函数的参数列表定义在 `(` 和 `)` 之间，返回值的类型则定义在之后的 `returntype (返回值类型)`处。声明一个参数的语法采用 **参数名** **参数类型** 的方式，任意多个参数采用类似 `(parameter1 type, parameter2 type) 即(参数1 参数1的类型,参数2 参数2的类型)`的形式指定。之后包含在 `{` 和 `}` 之间的代码，就是函数体。

函数中的参数列表和返回值并非是必须的，所以下面这个函数的声明也是有效的

```go
Copyfunc functionname() {  
    // 译注: 表示这个函数不需要输入参数，且没有返回值
}
```

## 示例函数

我们以写一个计算商品价格的函数为例，输入参数是单件商品的价格和商品的个数，两者的乘积为商品总价，作为函数的输出值。

```go
Copyfunc calculateBill(price int, no int) int {  
    var totalPrice = price * no // 商品总价 = 商品单价 * 数量
    return totalPrice // 返回总价
}
```

上述函数有两个整型的输入 `price` 和 `no`，返回值 `totalPrice` 为 `price` 和 `no` 的乘积，也是整数类型。

**如果有连续若干个参数，它们的类型一致，那么我们无须一一罗列，只需在最后一个参数后添加该类型。** 例如，`price int, no int` 可以简写为 `price, no int`，所以示例函数也可写成

```go
Copyfunc calculateBill(price, no int) int {  
    var totalPrice = price * no
    return totalPrice
}
```

现在我们已经定义了一个函数，我们要在代码中尝试着调用它。调用函数的语法为 `functionname(parameters)`。调用示例函数的方法如下：

```go
CopycalculateBill(10, 5)
```

完成了示例函数声明和调用后，我们就能写出一个完整的程序，并把商品总价打印在控制台上：

```go
Copypackage main

import (  
    "fmt"
)

func calculateBill(price, no int) int {  
    var totalPrice = price * no
    return totalPrice
}
func main() {  
    price, no := 90, 6 // 定义 price 和 no,默认类型为 int
    totalPrice := calculateBill(price, no)
    fmt.Println("Total price is", totalPrice) // 打印到控制台上
}
```

该程序在控制台上打印的结果为

```
CopyTotal price is 540
```

## 多返回值

Go 语言支持一个函数可以有多个返回值。我们来写个以矩形的长和宽为输入参数，计算并返回矩形面积和周长的函数 `rectProps`。矩形的面积是长度和宽度的乘积, 周长是长度和宽度之和的两倍。即：

- `面积 = 长 * 宽`
- `周长 = 2 * ( 长 + 宽 )`

```go
Copypackage main

import (  
    "fmt"
)

func rectProps(length, width float64)(float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}

func main() {  
    area, perimeter := rectProps(10.8, 5.6)
    fmt.Printf("Area %f Perimeter %f", area, perimeter) 
}
```

如果一个函数有多个返回值，那么这些返回值必须用 `(` 和 `)` 括起来。`func rectProps(length, width float64)(float64, float64)` 示例函数有两个 float64 类型的输入参数 `length` 和 `width`，并返回两个 float64 类型的值。该程序在控制台上打印结果为

```
CopyArea 60.480000 Perimeter 32.800000
```

## 命名返回值

从函数中可以返回一个命名值。一旦命名了返回值，可以认为这些值在函数第一行就被声明为变量了。

上面的 rectProps 函数也可用这个方式写成：

```go
Copyfunc rectProps(length, width float64)(area, perimeter float64) {  
    area = length * width
    perimeter = (length + width) * 2
    return // 不需要明确指定返回值，默认返回 area, perimeter 的值
}
```

请注意, 函数中的 return 语句没有显式返回任何值。由于 **area** 和 **perimeter** 在函数声明中指定为返回值, 因此当遇到 return 语句时, 它们将自动从函数返回。

## 空白符

**_** 在 Go 中被用作空白符，可以用作表示任何类型的任何值。

我们继续以 `rectProps` 函数为例，该函数计算的是面积和周长。假使我们只需要计算面积，而并不关心周长的计算结果，该怎么调用这个函数呢？这时，空白符 **_** 就上场了。

下面的程序我们只用到了函数 `rectProps` 的一个返回值 `area`

```go
Copypackage main

import (  
    "fmt"
)

func rectProps(length, width float64) (float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}
func main() {  
    area, _ := rectProps(10.8, 5.6) // 返回值周长被丢弃
    fmt.Printf("Area %f ", area)
}
```

> 在程序的 `area, _ := rectProps(10.8, 5.6)` 这一行，我们看到空白符 `_` 用来跳过不要的计算结果。

# 7. 包

### 什么是包，为什么使用包？

到目前为止，我们看到的 Go 程序都只有一个文件，文件里包含一个 main 函数和几个其他的函数。在实际中，这种把所有源代码编写在一个文件的方法并不好用。以这种方式编写，代码的重用和维护都会很困难。而包（Package）解决了这样的问题。

**包用于组织 Go 源代码，提供了更好的可重用性与可读性**。由于包提供了代码的封装，因此使得 Go 应用程序易于维护。

例如，假如我们正在开发一个 Go 图像处理程序，它提供了图像的裁剪、锐化、模糊和彩色增强等功能。一种组织程序的方式就是根据不同的特性，把代码放到不同的包中。比如裁剪可以是一个单独的包，而锐化是另一个包。这种方式的优点是，由于彩色增强可能需要一些锐化的功能，因此彩色增强的代码只需要简单地导入（我们会在随后讨论）锐化功能的包，就可以使用锐化的功能了。这样的方式使得代码易于重用。

我们会逐步构建一个计算矩形的面积和对角线的应用程序。

通过这个程序，我们会更好地理解包。

### main 函数和 main 包

所有可执行的 Go 程序都必须包含一个 main 函数。这个函数是程序运行的入口。main 函数应该放置于 main 包中。

**package packagename 这行代码指定了某一源文件属于一个包。它应该放在每一个源文件的第一行。**

下面开始为我们的程序创建一个 main 函数和 main 包。**在 Go 工作区内的 src 文件夹中创建一个文件夹，命名为 geometry**。在 `geometry` 文件夹中创建一个 `geometry.go` 文件。

在 geometry.go 中编写下面代码。

```go
Copy// geometry.go
package main 

import "fmt"

func main() {  
    fmt.Println("Geometrical shape properties")
}
```

`package main` 这一行指定该文件属于 main 包。`import "packagename"` 语句用于导入一个已存在的包。在这里我们导入了 `fmt` 包，包内含有 Println 方法。接下来是 main 函数，它会打印 `Geometrical shape properties`。

键入 `go install geometry`，编译上述程序。该命令会在 `geometry` 文件夹内搜索拥有 main 函数的文件。在这里，它找到了 `geometry.go`。接下来，它编译并产生一个名为 `geometry` （在 windows 下是 `geometry.exe`）的二进制文件，该二进制文件放置于工作区的 bin 文件夹。现在，工作区的目录结构会是这样：

```
Copysrc
    geometry
        gemometry.go
bin
    geometry
```

键入 `workspacepath/bin/geometry`，运行该程序。请用你自己的 Go 工作区来替换 `workspacepath`。这个命令会执行 bin 文件夹里的 `geometry` 二进制文件。你应该会输出 `Geometrical shape properties`。

### 创建自定义的包

我们将组织代码，使得所有与矩形有关的功能都放入 `rectangle` 包中。

我们会创建一个自定义包 `rectangle`，它有一个计算矩形的面积和对角线的函数。

**属于某一个包的源文件都应该放置于一个单独命名的文件夹里。按照 Go 的惯例，应该用包名命名该文件夹。**

因此，我们在 `geometry` 文件夹中，创建一个命名为 `rectangle` 的文件夹。在 `rectangle` 文件夹中，所有文件都会以 `package rectangle` 作为开头，因为它们都属于 rectangle 包。

在我们之前创建的 rectangle 文件夹中，再创建一个名为 `rectprops.go` 的文件，添加下列代码。

```go
Copy// rectprops.go
package rectangle

import "math"

func Area(len, wid float64) float64 {  
    area := len * wid
    return area
}

func Diagonal(len, wid float64) float64 {  
    diagonal := math.Sqrt((len * len) + (wid * wid))
    return diagonal
}
```

在上面的代码中，我们创建了两个函数用于计算 `Area` 和 `Diagonal`。矩形的面积是长和宽的乘积。矩形的对角线是长与宽平方和的平方根。`math` 包下面的 `Sqrt` 函数用于计算平方根。

注意到函数 Area 和 Diagonal 都是以大写字母开头的。这是有必要的，我们将会很快解释为什么需要这样做。

### 导入自定义包

为了使用自定义包，我们必须要先导入它。导入自定义包的语法为 `import path`。我们必须指定自定义包相对于工作区内 `src` 文件夹的相对路径。我们目前的文件夹结构是：

```
Copysrc
    geometry
        geometry.go
        rectangle
            rectprops.go
```

`import "geometry/rectangle"` 这一行会导入 rectangle 包。

在 `geometry.go` 里面添加下面的代码：

```go
Copy// geometry.go
package main 

import (  
    "fmt"
    "geometry/rectangle" // 导入自定义包
)

func main() {  
    var rectLen, rectWidth float64 = 6, 7
    fmt.Println("Geometrical shape properties")
    /*Area function of rectangle package used*/
    fmt.Printf("area of rectangle %.2f\n", rectangle.Area(rectLen, rectWidth))
    /*Diagonal function of rectangle package used*/
    fmt.Printf("diagonal of the rectangle %.2f ", rectangle.Diagonal(rectLen, rectWidth))
}
```

上面的代码导入了 `rectangle` 包，并调用了里面的 Area 和 Diagonal 函数，得到矩形的面积和对角线。Printf 内的格式说明符 `%.2f` 会将浮点数截断到小数点两位。应用程序的输出为：

```
CopyGeometrical shape properties  
area of rectangle 42.00  
diagonal of the rectangle 9.22
```

### 导出名字（Exported Names）

我们将 rectangle 包中的函数 Area 和 Diagonal 首字母大写。在 Go 中这具有特殊意义。在 Go 中，任何以大写字母开头的变量或者函数都是被导出的名字。其它包只能访问被导出的函数和变量。在这里，我们需要在 main 包中访问 Area 和 Diagonal 函数，因此会将它们的首字母大写。

在 `rectprops.go` 中，如果函数名从 `Area(len, wid float64)` 变为 `area(len, wid float64)`，并且在 `geometry.go` 中， `rectangle.Area(rectLen, rectWidth)` 变为 `rectangle.area(rectLen, rectWidth)`， 则该程序运行时，编译器会抛出错误 `geometry.go:11: cannot refer to unexported name rectangle.area`。因为如果想在包外访问一个函数，它应该首字母大写。

### init 函数

所有包都可以包含一个 `init` 函数。init 函数不应该有任何返回值类型和参数，在我们的代码中也不能显式地调用它。init 函数的形式如下：

```go
Copyfunc init() {  
}
```

init 函数可用于执行初始化任务，也可用于在开始执行之前验证程序的正确性。

包的初始化顺序如下：

1. 首先初始化包级别（Package Level）的变量
2. 紧接着调用 init 函数。包可以有多个 init 函数（在一个文件或分布于多个文件中），它们按照编译器解析它们的顺序进行调用。

如果一个包导入了另一个包，会先初始化被导入的包。

尽管一个包可能会被导入多次，但是它只会被初始化一次。

为了理解 init 函数，我们接下来对程序做了一些修改。

首先在 `rectprops.go` 文件中添加了一个 init 函数。

```go
Copy// rectprops.go
package rectangle

import "math"  
import "fmt"

/*
 * init function added
 */
func init() {  
    fmt.Println("rectangle package initialized")
}
func Area(len, wid float64) float64 {  
    area := len * wid
    return area
}

func Diagonal(len, wid float64) float64 {  
    diagonal := math.Sqrt((len * len) + (wid * wid))
    return diagonal
}
```

我们添加了一个简单的 init 函数，它仅打印 `rectangle package initialized`。

现在我们来修改 main 包。我们知道矩形的长和宽都应该大于 0，我们将在 `geometry.go` 中使用 init 函数和包级别的变量来检查矩形的长和宽。

修改 `geometry.go` 文件如下所示：

```go
Copy// geometry.gopackage main import (      "fmt"    "geometry/rectangle" // 导入自定义包    "log")/* * 1. 包级别变量*/var rectLen, rectWidth float64 = 6, 7 /**2. init 函数会检查长和宽是否大于0*/func init() {      println("main package initialized")    if rectLen < 0 {        log.Fatal("length is less than zero")    }    if rectWidth < 0 {        log.Fatal("width is less than zero")    }}func main() {      fmt.Println("Geometrical shape properties")    fmt.Printf("area of rectangle %.2f\n", rectangle.Area(rectLen, rectWidth))    fmt.Printf("diagonal of the rectangle %.2f ",rectangle.Diagonal(rectLen, rectWidth))}
```

我们对 `geometry.go` 做了如下修改：

1. 变量 **rectLen** 和 **rectWidth** 从 main 函数级别移到了包级别。
2. 添加了 init 函数。当 rectLen 或 rectWidth 小于 0 时，init 函数使用 **log.Fatal** 函数打印一条日志，并终止了程序。

main 包的初始化顺序为：

1. 首先初始化被导入的包。因此，首先初始化了 rectangle 包。
2. 接着初始化了包级别的变量 **rectLen** 和 **rectWidth**。
3. 调用 init 函数。
4. 最后调用 main 函数。

当运行该程序时，会有如下输出。

```
Copyrectangle package initialized  main package initialized  Geometrical shape properties  area of rectangle 42.00  diagonal of the rectangle 9.22
```

果然，程序会首先调用 rectangle 包的 init 函数，然后，会初始化包级别的变量 **rectLen** 和 **rectWidth**。接着调用 main 包里的 init 函数，该函数检查 rectLen 和 rectWidth 是否小于 0，如果条件为真，则终止程序。我们会在单独的教程里深入学习 if 语句。现在你可以认为 `if rectLen < 0` 能够检查 `rectLen` 是否小于 0，并且如果是，则终止程序。`rectWidth` 条件的编写也是类似的。在这里两个条件都为假，因此程序继续执行。最后调用了 main 函数。

让我们接着稍微修改这个程序来学习使用 init 函数。

将 `geometry.go` 中的 `var rectLen, rectWidth float64 = 6, 7` 改为 `var rectLen, rectWidth float64 = -6, 7`。我们把 `rectLen` 初始化为负数。

现在当运行程序时，会得到：

```
Copyrectangle package initialized  main package initialized  2017/04/04 00:28:20 length is less than zero
```

像往常一样， 会首先初始化 rectangle 包，然后是 main 包中的包级别的变量 rectLen 和 rectWidth。rectLen 为负数，因此当运行 init 函数时，程序在打印 `length is less than zero` 后终止。

### 使用空白标识符（Blank Identifier）

导入了包，却不在代码中使用它，这在 Go 中是非法的。当这么做时，编译器是会报错的。其原因是为了避免导入过多未使用的包，从而导致编译时间显著增加。将 `geometry.go` 中的代码替换为如下代码：

```go
Copy// geometry.go
package main 

import (
    "geometry/rectangle" // 导入自定的包
)
func main() {

}
```

上面的程序将会抛出错误 `geometry.go:6: imported and not used: "geometry/rectangle"`。

然而，在程序开发的活跃阶段，又常常会先导入包，而暂不使用它。遇到这种情况就可以使用空白标识符 `_`。

下面的代码可以避免上述程序的错误：

```go
Copypackage main

import (  
    "geometry/rectangle" 
)

var _ = rectangle.Area // 错误屏蔽器

func main() {

}
```

`var _ = rectangle.Area` 这一行屏蔽了错误。我们应该了解这些错误屏蔽器（Error Silencer）的动态，在程序开发结束时就移除它们，包括那些还没有使用过的包。由此建议在 import 语句下面的包级别范围中写上错误屏蔽器。

有时候我们导入一个包，只是为了确保它进行了初始化，而无需使用包中的任何函数或变量。例如，我们或许需要确保调用了 rectangle 包的 init 函数，而不需要在代码中使用它。这种情况也可以使用空白标识符，如下所示。

```go
Copypackage main import (    _ "geometry/rectangle" )func main() {}
```

运行上面的程序，会输出 `rectangle package initialized`。尽管在所有代码里，我们都没有使用这个包，但还是成功初始化了它。

# 8. if-else 语句

if 是条件语句。if 语句的语法是

```go
Copyif condition {  
}
```

如果 `condition` 为真，则执行 `{` 和 `}` 之间的代码。

不同于其他语言，例如 C 语言，Go 语言里的 `{ }` 是必要的，即使在 `{ }` 之间只有一条语句。

if 语句还有可选的 `else if` 和 `else` 部分。

```go
Copyif condition {  
} else if condition {
} else {
}
```

if-else 语句之间可以有任意数量的 `else if`。条件判断顺序是从上到下。如果 `if` 或 `else if` 条件判断的结果为真，则执行相应的代码块。 如果没有条件为真，则 `else` 代码块被执行。

让我们编写一个简单的程序来检测一个数字是奇数还是偶数。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    }  else {
        fmt.Println("the number is odd")
    }
}
```

`if num％2 == 0` 语句检测 num 取 2 的余数是否为零。 如果是为零则打印输出 "the number is even"，如果不为零则打印输出 "the number is odd"。在上面的这个程序中，打印输出的是 `the number is even`。

`if` 还有另外一种形式，它包含一个 `statement` 可选语句部分，该组件在条件判断之前运行。它的语法是

```go
Copyif statement; condition {  
}
```

让我们重写程序，使用上面的语法来查找数字是偶数还是奇数。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    if num := 10; num % 2 == 0 { //checks if number is even
        fmt.Println(num,"is even") 
    }  else {
        fmt.Println(num,"is odd")
    }
}
```

在上面的程序中，`num` 在 `if` 语句中进行初始化，`num` 只能从 `if` 和 `else` 中访问。也就是说 `num` 的范围仅限于 `if` `else` 代码块。如果我们试图从其他外部的 `if` 或者 `else` 访问 `num`,编译器会不通过。

让我们再写一个使用 `else if` 的程序。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    num := 99
    if num <= 50 {
        fmt.Println("number is less than or equal to 50")
    } else if num >= 51 && num <= 100 {
        fmt.Println("number is between 51 and 100")
    } else {
        fmt.Println("number is greater than 100")
    }

}
```

在上面的程序中，如果 `else if num >= 51 && num <= 100` 为真，程序将输出 `number is between 51 and 100`。

### 一个注意点

`else` 语句应该在 `if` 语句的大括号 `}` 之后的同一行中。如果不是，编译器会不通过。

让我们通过以下程序来理解它。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num % 2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    }  
    else {
        fmt.Println("the number is odd")
    }
}
```

在上面的程序中，`else` 语句不是从 `if` 语句结束后的 `}` 同一行开始。而是从下一行开始。这是不允许的。如果运行这个程序，编译器会输出错误，

```
Copymain.go:12:5: syntax error: unexpected else, expecting }
```

出错的原因是 Go 语言的分号是自动插入。

在 Go 语言规则中，它指定在 `}` 之后插入一个分号，如果这是该行的最终标记。因此，在if语句后面的 `}` 会自动插入一个分号。

实际上我们的程序变成了

```go
Copyif num%2 == 0 {  
      fmt.Println("the number is even") 
};  //semicolon inserted by Go
else {  
      fmt.Println("the number is odd")
}
```

分号插入之后。从上面代码片段可以看出第三行插入了分号。

由于 `if{…} else {…}` 是一个单独的语句，它的中间不应该出现分号。因此，需要将 `else` 语句放置在 `}` 之后处于同一行中。

我已经重写了程序，将 else 语句移动到 if 语句结束后 `}` 的后面，以防止分号的自动插入。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    num := 10
    if num%2 == 0 { //checks if number is even
        fmt.Println("the number is even") 
    } else {
        fmt.Println("the number is odd")
    }
}
```

现在编译器会很开心，我们也一样 ?。

# 9. 循环

循环语句是用来重复执行某一段代码。

`for` 是 Go 语言唯一的循环语句。Go 语言中并没有其他语言比如 C 语言中的 `while` 和 `do while` 循环。

## for 循环语法

```go
Copyfor initialisation; condition; post {  
}
```

初始化语句只执行一次。循环初始化后，将检查循环条件。如果条件的计算结果为 `true` ，则 `{}` 内的循环体将执行，接着执行 post 语句。post 语句将在每次成功循环迭代后执行。在执行 post 语句后，条件将被再次检查。如果为 `true`, 则循环将继续执行，否则 for 循环将终止。（译注：这是典型的 for 循环三个表达式，第一个为初始化表达式或赋值语句；第二个为循环条件判定表达式；第三个为循环变量修正表达式，即此处的 post ）

这三个组成部分，即初始化，条件和 post 都是可选的。让我们看一个例子来更好地理解循环。

## 例子

让我们用 `for` 循环写一个打印出从 1 到 10 的程序。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        fmt.Printf(" %d",i)
    }
}
```

在上面的程序中，i 变量被初始化为 1。条件语句会检查 i 是否小于 10。如果条件成立，i 就会被打印出来，否则循环就会终止。循环语句会在每一次循环完成后自增 1。一旦 i 变得比 10 要大，循环中止。

上面的程序会打印出 `1 2 3 4 5 6 7 8 9 10` 。

在 `for` 循环中声明的变量只能在循环体内访问，因此 i 不能够在循环体外访问。

## break

`break` 语句用于在完成正常执行之前突然终止 for 循环，之后程序将会在 for 循环下一行代码开始执行。

让我们写一个从 1 打印到 5 并且使用 `break` 跳出循环的程序。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i > 5 {
            break //loop is terminated if i > 5
        }
        fmt.Printf("%d ", i)
    }
    fmt.Printf("\nline after for loop")
}
```

在上面的程序中，在循环过程中 i 的值会被判断。如果 i 的值大于 5 然后 `break` 语句就会执行，循环就会被终止。打印语句会在 `for` 循环结束后执行，上面程序会输出为

```
Copy1 2 3 4 5  
line after for loop
```

## continue

`continue` 语句用来跳出 `for` 循环中当前循环。在 `continue` 语句后的所有的 `for` 循环语句都不会在本次循环中执行。循环体会在一下次循环中继续执行。

让我们写一个打印出 1 到 10 并且使用 `continue` 的程序。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    for i := 1; i <= 10; i++ {
        if i%2 == 0 {
            continue
        }
        fmt.Printf("%d ", i)
    }
}
```

在上面的程序中，这行代码 `if i%2==0` 会判断 i 除以 2 的余数是不是 0，如果是 0，这个数字就是偶数然后执行 `continue` 语句，从而控制程序进入下一个循环。因此在 `continue` 后面的打印语句不会被调用而程序会进入一下个循环。上面程序会输出 `1 3 5 7 9`。

## 更多例子

让我们写更多的代码来演示 `for` 循环的多样性吧

下面这个程序打印出从 0 到 10 所有的偶数。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    i := 0
    for ;i <= 10; { // initialisation and post are omitted
        fmt.Printf("%d ", i)
        i += 2
    }
}
```

正如我们已经知道的那样，`for` 循环的三部分，初始化语句、条件语句、post 语句都是可选的。在上面的程序中，初始化语句和 post 语句都被省略了。i 在 `for` 循环外被初始化成了 0。只要 `i<=10` 循环就会被执行。在循环中，i 以 2 的增量自增。上面的程序会输出 `0 2 4 6 8 10`。

上面程序中 `for` 循环中的分号也可以省略。这个格式的 `for` 循环可以看作是二选一的 `for while` 循环。上面的程序可以被重写成：

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    i := 0
    for i <= 10 { //semicolons are ommitted and only condition is present
        fmt.Printf("%d ", i)
        i += 2
    }
}
```

在 `for` 循环中可以声明和操作多个变量。让我们写一个使用声明多个变量来打印下面序列的程序。

```
Copy10 * 1 = 10  
11 * 2 = 22  
12 * 3 = 36  
13 * 4 = 52  
14 * 5 = 70  
15 * 6 = 90  
16 * 7 = 112  
17 * 8 = 136  
18 * 9 = 162  
19 * 10 = 190
package main

import (  
    "fmt"
)

func main() {  
    for no, i := 10, 1; i <= 10 && no <= 19; i, no = i+1, no+1 { //multiple initialisation and increment
        fmt.Printf("%d * %d = %d\n", no, i, no*i)
    }

}
```

在上面的程序中 `no` 和 `i` 被声明然后分别被初始化为 10 和 1 。在每一次循环结束后 `no` 和 `i` 都自增 1 。布尔型操作符 `&&` 被用来确保 i 小于等于 10 并且 `no` 小于等于 19 。

## 无限循环

无限循环的语法是：

```go
Copyfor {  
}
```

下一个程序就会一直打印`Hello World`不会停止。

```go
Copypackage main

import "fmt"

func main() {  
    for {
        fmt.Println("Hello World")
    }
}
```

在你本地系统上运行，来无限的打印 “Hello World” 。

这里还有一个 `range` 结构，它可以被用来在 `for` 循环中操作数组对象。当我们学习数组时我们会补充这方面内容。

# 10. switch 语句

switch 是一个条件语句，用于将表达式的值与可能匹配的选项列表进行比较，并根据匹配情况执行相应的代码块。它可以被认为是替代多个 `if else` 子句的常用方式。

看代码比文字更容易理解。让我们从一个简单的例子开始，它将把一个手指的编号作为输入，然后输出该手指对应的名字。比如 0 是拇指，1 是食指等等。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    finger := 4
    switch finger {
    case 1:
        fmt.Println("Thumb")
    case 2:
        fmt.Println("Index")
    case 3:
        fmt.Println("Middle")
    case 4:
        fmt.Println("Ring")
    case 5:
        fmt.Println("Pinky")

    }
}
```

在上述程序中，`switch finger` 将 `finger` 的值与每个 `case` 语句进行比较。通过从上到下对每一个值进行对比，并执行与选项值匹配的第一个逻辑。在上述样例中， `finger` 值为 4，因此打印的结果是 `Ring` 。

在选项列表中，`case` 不允许出现重复项。如果您尝试运行下面的程序，编译器会报这样的错误: `main.go：18：2：在tmp / sandbox887814166 / main.go：16：7`

```go
Copypackage main

import (
    "fmt"
)

func main() {
    finger := 4
    switch finger {
    case 1:
        fmt.Println("Thumb")
    case 2:
        fmt.Println("Index")
    case 3:
        fmt.Println("Middle")
    case 4:
        fmt.Println("Ring")
    case 4://重复项
        fmt.Println("Another Ring")
    case 5:
        fmt.Println("Pinky")

    }
}
```

## 默认情况（Default Case）

我们每个人一只手只有 5 个手指。如果我们输入了不正确的手指编号会发生什么？这个时候就应该是属于默认情况。当其他情况都不匹配时，将运行默认情况。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    switch finger := 8; finger {
    case 1:
        fmt.Println("Thumb")
    case 2:
        fmt.Println("Index")
    case 3:
        fmt.Println("Middle")
    case 4:
        fmt.Println("Ring")
    case 5:
        fmt.Println("Pinky")
    default: // 默认情况
        fmt.Println("incorrect finger number")
    }
}
```

在上述程序中 `finger` 的值是 8，它不符合其中任何情况，因此会打印 `incorrect finger number`。default 不一定只能出现在 switch 语句的最后，它可以放在 switch 语句的任何地方。

您可能也注意到我们稍微改变了 `finger` 变量的声明方式。`finger` 声明在了 switch 语句内。在表达式求值之前，switch 可以选择先执行一个语句。在这行 `switch finger：= 8; finger` 中， 先声明了`finger` 变量，随即在表达式中使用了它。在这里，`finger` 变量的作用域仅限于这个 switch 内。

## 多表达式判断

通过用逗号分隔，可以在一个 case 中包含多个表达式。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    letter := "i"
    switch letter {
    case "a", "e", "i", "o", "u": // 一个选项多个表达式
        fmt.Println("vowel")
    default:
        fmt.Println("not a vowel")
    }
}
```

在 `case "a","e","i","o","u":` 这一行中，列举了所有的元音。只要匹配该项，则将输出 `vowel`。

## 无表达式的 switch

在 switch 语句中，表达式是可选的，可以被省略。如果省略表达式，则表示这个 switch 语句等同于 `switch true`，并且每个 `case` 表达式都被认定为有效，相应的代码块也会被执行。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    num := 75
    switch { // 表达式被省略了
    case num >= 0 && num <= 50:
        fmt.Println("num is greater than 0 and less than 50")
    case num >= 51 && num <= 100:
        fmt.Println("num is greater than 51 and less than 100")
    case num >= 101:
        fmt.Println("num is greater than 100")
    }

}
```

在上述代码中，switch 中缺少表达式，因此默认它为 true，true 值会和每一个 case 的求值结果进行匹配。`case num >= 51 && <= 100:` 为 true，所以程序输出 `num is greater than 51 and less than 100`。这种类型的 switch 语句可以替代多个 `if else` 子句。

## Fallthrough 语句

在 Go 中，每执行完一个 case 后，会从 switch 语句中跳出来，不再做后续 case 的判断和执行。使用 `fallthrough` 语句可以在已经执行完成的 case 之后，把控制权转移到下一个 case 的执行代码中。

让我们写一个程序来理解 fallthrough。我们的程序将检查输入的数字是否小于 50、100 或 200。例如我们输入 75，程序将输出`75 is lesser than 100` 和 `75 is lesser than 200`。我们用 fallthrough 来实现了这个功能。

```go
Copypackage main

import (
    "fmt"
)

func number() int {
    num := 15 * 5 
    return num
}

func main() {

    switch num := number(); { // num is not a constant
    case num < 50:
        fmt.Printf("%d is lesser than 50\n", num)
        fallthrough
    case num < 100:
        fmt.Printf("%d is lesser than 100\n", num)
        fallthrough
    case num < 200:
        fmt.Printf("%d is lesser than 200", num)
    }

}
```

switch 和 case 的表达式不一定是常量。它们也可以在运行过程中通过计算得到。在上面的程序中，num 被初始化为函数 `number()` 的返回值。程序运行到 switch 中时，会计算出 case 的值。`case num < 100：` 的结果为 true，所以程序输出 `75 is lesser than 100`。当执行到下一句 `fallthrough` 时，程序控制直接跳转到下一个 case 的第一个执行逻辑中，所以打印出 `75 is lesser than 200`。最后这个程序的输出会是

```
Copy75 is lesser than 100  
75 is lesser than 200
```

**fallthrough 语句应该是 case 子句的最后一个语句。如果它出现在了 case 语句的中间，编译器将会报错：fallthrough statement out of place**11. 数组和切片

## 数组

数组是同一类型元素的集合。例如，整数集合 5,8,9,79,76 形成一个数组。Go 语言中不允许混合不同类型的元素，例如包含字符串和整数的数组。（译者注：当然，如果是 interface{} 类型数组，可以包含任意类型）

### 数组的声明

一个数组的表示形式为 `[n]T`。`n` 表示数组中元素的数量，`T` 代表每个元素的类型。元素的数量 `n` 也是该类型的一部分（稍后我们将详细讨论这一点）。

可以使用不同的方式来声明数组，让我们一个一个的来看。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    var a [3]int //int array with length 3
    fmt.Println(a)
}
```

**var a[3]int** 声明了一个长度为 3 的整型数组。**数组中的所有元素都被自动赋值为数组类型的零值。** 在这种情况下，`a` 是一个整型数组，因此 `a` 的所有元素都被赋值为 `0`，即 int 型的零值。运行上述程序将 **输出** `[0 0 0]`。

数组的索引从 `0` 开始到 `length - 1` 结束。让我们给上面的数组赋值。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    var a [3]int //int array with length 3
    a[0] = 12 // array index starts at 0
    a[1] = 78
    a[2] = 50
    fmt.Println(a)
}
```

a[0] 将值赋给数组的第一个元素。该程序将 **输出** `[12 78 50]`。

让我们使用 **简略声明** 来创建相同的数组。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    a := [3]int{12, 78, 50} // short hand declaration to create array
    fmt.Println(a)
}
```

上面的程序将会打印相同的 **输出** `[12 78 50]`。

在简略声明中，不需要将数组中所有的元素赋值。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    a := [3]int{12} 
    fmt.Println(a)
}
```

在上述程序中的第 8 行 `a := [3]int{12}` 声明一个长度为 3 的数组，但只提供了一个值 `12`，剩下的 2 个元素自动赋值为 `0`。这个程序将**输出** `[12 0 0]`。

你甚至可以忽略声明数组的长度，并用 `...` 代替，让编译器为你自动计算长度，这在下面的程序中实现。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    a := [...]int{12, 78, 50} // ... makes the compiler determine the length
    fmt.Println(a)
}
```

**数组的大小是类型的一部分**。因此 `[5]int` 和 `[25]int` 是不同类型。数组不能调整大小，不要担心这个限制，因为 `slices` 的存在能解决这个问题。

```go
Copypackage main

func main() {
    a := [3]int{5, 78, 8}
    var b [5]int
    b = a // not possible since [3]int and [5]int are distinct types
}
```

在上述程序的第 6 行中, 我们试图将类型 `[3]int` 的变量赋给类型为 `[5]int` 的变量，这是不允许的，因此编译器将抛出错误 main.go:6: cannot use a (type [3]int) as type [5]int in assignment。

### 数组是值类型

Go 中的数组是值类型而不是引用类型。这意味着当数组赋值给一个新的变量时，该变量会得到一个原始数组的一个副本。如果对新变量进行更改，则不会影响原始数组。

```go
Copypackage main

import "fmt"

func main() {
    a := [...]string{"USA", "China", "India", "Germany", "France"}
    b := a // a copy of a is assigned to b
    b[0] = "Singapore"
    fmt.Println("a is ", a)
    fmt.Println("b is ", b) 
}
```

在上述程序的第 7 行，`a` 的副本被赋给 `b`。在第 8 行中，`b` 的第一个元素改为 `Singapore`。这不会在原始数组 `a` 中反映出来。该程序将 **输出**,

```
Copya is [USA China India Germany France]  
b is [Singapore China India Germany France]
```

同样，当数组作为参数传递给函数时，它们是按值传递，而原始数组保持不变。

```go
Copypackage main

import "fmt"

func changeLocal(num [5]int) {
    num[0] = 55
    fmt.Println("inside function ", num)
}
func main() {
    num := [...]int{5, 6, 7, 8, 8}
    fmt.Println("before passing to function ", num)
    changeLocal(num) //num is passed by value
    fmt.Println("after passing to function ", num)
}
```

在上述程序的 13 行中, 数组 `num` 实际上是通过值传递给函数 `changeLocal`，数组不会因为函数调用而改变。这个程序将输出,

```
Copybefore passing to function  [5 6 7 8 8]
inside function  [55 6 7 8 8]
after passing to function  [5 6 7 8 8]
```

### 数组的长度

通过将数组作为参数传递给 `len` 函数，可以得到数组的长度。

```go
Copypackage main

import "fmt"

func main() {
    a := [...]float64{67.7, 89.8, 21, 78}
    fmt.Println("length of a is",len(a))
}
```

上面的程序输出为 `length of a is 4`。

### 使用 range 迭代数组

`for` 循环可用于遍历数组中的元素。

```go
Copypackage main

import "fmt"

func main() {
    a := [...]float64{67.7, 89.8, 21, 78}
    for i := 0; i < len(a); i++ { // looping from 0 to the length of the array
        fmt.Printf("%d th element of a is %.2f\n", i, a[i])
    }
}
```

上面的程序使用 `for` 循环遍历数组中的元素，从索引 `0` 到 `length of the array - 1`。这个程序运行后打印出，

```
Copy0 th element of a is 67.70  
1 th element of a is 89.80  
2 th element of a is 21.00  
3 th element of a is 78.00
```

Go 提供了一种更好、更简洁的方法，通过使用 `for` 循环的 **range** 方法来遍历数组。`range` 返回索引和该索引处的值。让我们使用 range 重写上面的代码。我们还可以获取数组中所有元素的总和。

```go
Copypackage main

import "fmt"

func main() {
    a := [...]float64{67.7, 89.8, 21, 78}
    sum := float64(0)
    for i, v := range a {//range returns both the index and value
        fmt.Printf("%d the element of a is %.2f\n", i, v)
        sum += v
    }
    fmt.Println("\nsum of all elements of a",sum)
}
```

上述程序的第 8 行 `for i, v := range a` 利用的是 for 循环 range 方式。 它将返回索引和该索引处的值。 我们打印这些值，并计算数组 `a` 中所有元素的总和。 程序的 **输出是**，

```
Copy0 the element of a is 67.70
1 the element of a is 89.80
2 the element of a is 21.00
3 the element of a is 78.00

sum of all elements of a 256.5
```

如果你只需要值并希望忽略索引，则可以通过用 `_` 空白标识符替换索引来执行。

```go
Copyfor _, v := range a { // ignores index  
}
```

上面的 for 循环忽略索引，同样值也可以被忽略。

### 多维数组

到目前为止我们创建的数组都是一维的，Go 语言可以创建多维数组。

```go
Copypackage main

import (
    "fmt"
)

func printarray(a [3][2]string) {
    for _, v1 := range a {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}

func main() {
    a := [3][2]string{
        {"lion", "tiger"},
        {"cat", "dog"},
        {"pigeon", "peacock"}, // this comma is necessary. The compiler will complain if you omit this comma
    }
    printarray(a)
    var b [3][2]string
    b[0][0] = "apple"
    b[0][1] = "samsung"
    b[1][0] = "microsoft"
    b[1][1] = "google"
    b[2][0] = "AT&T"
    b[2][1] = "T-Mobile"
    fmt.Printf("\n")
    printarray(b)
}
```

在上述程序的第 17 行，用简略语法声明一个二维字符串数组 a 。20 行末尾的逗号是必需的。这是因为根据 Go 语言的规则自动插入分号。至于为什么这是必要的，如果你想了解更多，请阅读https://golang.org/doc/effective_go.html#semicolons。

另外一个二维数组 b 在 23 行声明，字符串通过每个索引一个一个添加。这是另一种初始化二维数组的方法。

第 7 行的 printarray 函数使用两个 range 循环来打印二维数组的内容。上述程序的 **输出是**

```
Copylion tiger
cat dog
pigeon peacock

apple samsung
microsoft google
AT&T T-Mobile
```

这就是数组，尽管数组看上去似乎足够灵活，但是它们具有固定长度的限制，不可能增加数组的长度。这就要用到 **切片** 了。事实上，在 Go 中，切片比传统数组更常见。

## 切片

切片是由数组建立的一种方便、灵活且功能强大的包装（Wrapper）。切片本身不拥有任何数据。它们只是对现有数组的引用。

### 创建一个切片

带有 T 类型元素的切片由 `[]T` 表示

```go
Copypackage main

import (
    "fmt"
)

func main() {
    a := [5]int{76, 77, 78, 79, 80}
    var b []int = a[1:4] // creates a slice from a[1] to a[3]
    fmt.Println(b)
}
```

使用语法 `a[start:end]` 创建一个从 `a` 数组索引 `start` 开始到 `end - 1` 结束的切片。因此，在上述程序的第 9 行中, `a[1:4]` 从索引 1 到 3 创建了 `a` 数组的一个切片表示。因此, 切片 `b` 的值为 `[77 78 79]`。

让我们看看另一种创建切片的方法。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    c := []int{6, 7, 8} // creates and array and returns a slice reference
    fmt.Println(c)
}
```

在上述程序的第 9 行，`c：= [] int {6，7，8}` 创建一个有 3 个整型元素的数组，并返回一个存储在 c 中的切片引用。

### 切片的修改

切片自己不拥有任何数据。它只是底层数组的一种表示。对切片所做的任何修改都会反映在底层数组中。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before", darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after", darr)
}
```

在上述程序的第 9 行，我们根据数组索引 2,3,4 创建一个切片 `dslice`。for 循环将这些索引中的值逐个递增。当我们使用 for 循环打印数组时，我们可以看到对切片的更改反映在数组中。该程序的输出是

```
Copyarray before [57 89 90 82 100 78 67 69 59]  
array after [57 89 91 83 101 78 67 69 59]
```

当多个切片共用相同的底层数组时，每个切片所做的更改将反映在数组中。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    numa := [3]int{78, 79 ,80}
    nums1 := numa[:] // creates a slice which contains all elements of the array
    nums2 := numa[:]
    fmt.Println("array before change 1", numa)
    nums1[0] = 100
    fmt.Println("array after modification to slice nums1", numa)
    nums2[1] = 101
    fmt.Println("array after modification to slice nums2", numa)
}
```

在 9 行中，`numa [:]` 缺少开始和结束值。开始和结束的默认值分别为 `0` 和 `len (numa)`。两个切片 `nums1` 和 `nums2` 共享相同的数组。该程序的输出是

```
Copyarray before change 1 [78 79 80]  
array after modification to slice nums1 [100 79 80]  
array after modification to slice nums2 [100 101 80]
```

从输出中可以清楚地看出，当切片共享同一个数组时，每个所做的修改都会反映在数组中。

### 切片的长度和容量

切片的长度是切片中的元素数。**切片的容量是从创建切片索引开始的底层数组中元素数。**

让我们写一段代码来更好地理解这点。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d", len(fruitslice), cap(fruitslice)) // length of is 2 and capacity is 6
}
```

在上面的程序中，`fruitslice` 是从 `fruitarray` 的索引 1 和 2 创建的。 因此，`fruitlice` 的长度为 `2`。

`fruitarray` 的长度是 7。`fruiteslice` 是从 `fruitarray` 的索引 `1` 创建的。因此, `fruitslice` 的容量是从 `fruitarray` 索引为 `1` 开始，也就是说从 `orange` 开始，该值是 `6`。因此, `fruitslice` 的容量为 6。该[程序]输出切片的 **长度为 2 容量为 6** 。

切片可以重置其容量。任何超出这一点将导致程序运行时抛出错误。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    fruitarray := [...]string{"apple", "orange", "grape", "mango", "water melon", "pine apple", "chikoo"}
    fruitslice := fruitarray[1:3]
    fmt.Printf("length of slice %d capacity %d\n", len(fruitslice), cap(fruitslice)) // length of is 2 and capacity is 6
    fruitslice = fruitslice[:cap(fruitslice)] // re-slicing furitslice till its capacity
    fmt.Println("After re-slicing length is",len(fruitslice), "and capacity is",cap(fruitslice))
}
```

在上述程序的第 11 行中，`fruitslice` 的容量是重置的。以上程序输出为，

```
Copylength of slice 2 capacity 6 
After re-slicing length is 6 and capacity is 6
```

### 使用 make 创建一个切片

func make（[]T，len，cap）[]T 通过传递类型，长度和容量来创建切片。容量是可选参数, 默认值为切片长度。make 函数创建一个数组，并返回引用该数组的切片。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    i := make([]int, 5, 5)
    fmt.Println(i)
}
```

使用 make 创建切片时默认情况下这些值为零。上述程序的输出为 `[0 0 0 0 0]`。

### 追加切片元素

正如我们已经知道数组的长度是固定的，它的长度不能增加。 切片是动态的，使用 `append` 可以将新元素追加到切片上。append 函数的定义是 `func append（s[]T，x ... T）[]T`。

**x ... T** 在函数定义中表示该函数接受参数 x 的个数是可变的。这些类型的函数被称为[可变函数]。

有一个问题可能会困扰你。如果切片由数组支持，并且数组本身的长度是固定的，那么切片如何具有动态长度。以及内部发生了什么，当新的元素被添加到切片时，会创建一个新的数组。现有数组的元素被复制到这个新数组中，并返回这个新数组的新切片引用。现在新切片的容量是旧切片的两倍。下面的程序会让你清晰理解。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    cars := []string{"Ferrari", "Honda", "Ford"}
    fmt.Println("cars:", cars, "has old length", len(cars), "and capacity", cap(cars)) // capacity of cars is 3
    cars = append(cars, "Toyota")
    fmt.Println("cars:", cars, "has new length", len(cars), "and capacity", cap(cars)) // capacity of cars is doubled to 6
}
```

在上述程序中，`cars` 的容量最初是 3。在第 10 行，我们给 cars 添加了一个新的元素，并把 `append(cars, "Toyota")` 返回的切片赋值给 cars。现在 cars 的容量翻了一番，变成了 6。上述程序的输出是

```
Copycars: [Ferrari Honda Ford] has old length 3 and capacity 3  
cars: [Ferrari Honda Ford Toyota] has new length 4 and capacity 6
```

切片类型的零值为 `nil`。一个 `nil` 切片的长度和容量为 0。可以使用 append 函数将值追加到 `nil` 切片。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    var names []string //zero value of a slice is nil
    if names == nil {
        fmt.Println("slice is nil going to append")
        names = append(names, "John", "Sebastian", "Vinay")
        fmt.Println("names contents:",names)
    }
}
```

在上面的程序 `names` 是 nil，我们已经添加 3 个字符串给 `names`。该程序的输出是

```
Copyslice is nil going to append  
names contents: [John Sebastian Vinay]
```

也可以使用 `...` 运算符将一个切片添加到另一个切片。 你可以在[可变参数函数]教程中了解有关此运算符的更多信息。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    veggies := []string{"potatoes", "tomatoes", "brinjal"}
    fruits := []string{"oranges", "apples"}
    food := append(veggies, fruits...)
    fmt.Println("food:",food)
}
```

在上述程序的第 10 行，food 是通过 append(veggies, fruits...) 创建。程序的输出为 `food: [potatoes tomatoes brinjal oranges apples]`。

### 切片的函数传递

我们可以认为，切片在内部可由一个结构体类型表示。这是它的表现形式，

```go
Copytype slice struct {  
    Length        int
    Capacity      int
    ZerothElement *byte
}
```

切片包含长度、容量和指向数组第零个元素的指针。当切片传递给函数时，即使它通过值传递，指针变量也将引用相同的底层数组。因此，当切片作为参数传递给函数时，函数内所做的更改也会在函数外可见。让我们写一个程序来检查这点。

```go
Copypackage main

import (
    "fmt"
)

func subtactOne(numbers []int) {
    for i := range numbers {
        numbers[i] -= 2
    }
}
func main() {
    nos := []int{8, 7, 6}
    fmt.Println("slice before function call", nos)
    subtactOne(nos)                               // function modifies the slice
    fmt.Println("slice after function call", nos) // modifications are visible outside
}
```

上述程序的行号 17 中，调用函数将切片中的每个元素递减 2。在函数调用后打印切片时，这些更改是可见的。如果你还记得，这是不同于数组的，对于函数中一个数组的变化在函数外是不可见的。上述[程序]的输出是，

```
Copyarray before function call [8 7 6]  
array after function call [6 5 4]
```

### 多维切片

类似于数组，切片可以有多个维度。

```go
Copypackage main

import (
    "fmt"
)

func main() {  
     pls := [][]string {
            {"C", "C++"},
            {"JavaScript"},
            {"Go", "Rust"},
            }
    for _, v1 := range pls {
        for _, v2 := range v1 {
            fmt.Printf("%s ", v2)
        }
        fmt.Printf("\n")
    }
}
```

程序的输出为，

```
CopyC C++  
JavaScript  
Go Rust
```

### 内存优化

切片持有对底层数组的引用。只要切片在内存中，数组就不能被垃圾回收。在内存管理方面，这是需要注意的。让我们假设我们有一个非常大的数组，我们只想处理它的一小部分。然后，我们由这个数组创建一个切片，并开始处理切片。这里需要重点注意的是，在切片引用时数组仍然存在内存中。

一种解决方法是使用 [copy] 函数 `func copy(dst，src[]T)int` 来生成一个切片的副本。这样我们可以使用新的切片，原始数组可以被垃圾回收。

```go
Copypackage main

import (
    "fmt"
)

func countries() []string {
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
    neededCountries := countries[:len(countries)-2]
    countriesCpy := make([]string, len(neededCountries))
    copy(countriesCpy, neededCountries) //copies neededCountries to countriesCpy
    return countriesCpy
}
func main() {
    countriesNeeded := countries()
    fmt.Println(countriesNeeded)
}
```

在上述程序的第 9 行，`neededCountries := countries[:len(countries)-2` 创建一个去掉尾部 2 个元素的切片 `countries`，在上述程序的 11 行，将 `neededCountries` 复制到 `countriesCpy` 同时在函数的下一行返回 countriesCpy。现在 `countries` 数组可以被垃圾回收, 因为 `neededCountries` 不再被引用。

# 12. 可变参数函数

## 什么是可变参数函数

可变参数函数是一种参数个数可变的函数。

## 语法

如果函数最后一个参数被记作 `...T` ，这时函数可以接受任意个 `T` 类型参数作为最后一个参数。

请注意只有函数的最后一个参数才允许是可变的。

## 通过一些例子理解可变参数函数如何工作

你是否曾经想过 append 函数是如何将任意个参数值加入到切片中的。这样 append 函数可以接受不同数量的参数。

```go
Copyfunc append(slice []Type, elems ...Type) []Type
```

上面是 append 函数的定义。在定义中 elems 是可变参数。这样 append 函数可以接受可变化的参数。

让我们创建一个我们自己的可变参数函数。我们将写一段简单的程序，在输入的整数列表里查找某个整数是否存在。

```go
Copypackage main

import (
    "fmt"
)

func find(num int, nums ...int) {
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {
    find(89, 89, 90, 95)
    find(45, 56, 67, 45, 90, 109)
    find(78, 38, 56, 98)
    find(87)
}
```

在上面程序中 `func find(num int, nums ...int)` 中的 `nums` 可接受任意数量的参数。在 find 函数中，参数 `nums` 相当于一个整型切片。

**可变参数函数的工作原理是把可变参数转换为一个新的切片。以上面程序中的第 22 行为例，find 函数中的可变参数是 89，90，95 。 find 函数接受一个 int 类型的可变参数。因此这三个参数被编译器转换为一个 int 类型切片 int []int{89, 90, 95} 然后被传入 find函数。**

在第 10 行， `for` 循环遍历 `nums` 切片,如果 `num` 在切片中，则打印 `num` 的位置。如果 `num` 不在切片中,则打印提示未找到该数字。

上面代码的输出值如下,

```
Copytype of nums is []int
89 found at index 0 in [89 90 95]

type of nums is []int
45 found at index 2 in [56 67 45 90 109]

type of nums is []int
78 not found in  [38 56 98]

type of nums is []int
87 not found in  []
```

在上面程序的第 25 行，find 函数仅有一个参数。我们没有给可变参数 `nums ...int` 传入任何参数。这也是合法的，在这种情况下 `nums` 是一个长度和容量为 0 的 `nil` 切片。

## 给可变参数函数传入切片

下面例子中，我们给可变参数函数传入一个切片，看看会发生什么。

```go
Copypackage main

import (
    "fmt"
)

func find(num int, nums ...int) {
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {
    nums := []int{89, 90, 95}
    find(89, nums)
}
```

在第 23 行中，我们将一个切片传给一个可变参数函数。

这种情况下无法通过编译，编译器报出错误 `main.go:23: cannot use nums (type []int) as type int in argument to find` 。

为什么无法工作呢？原因很直接，`find` 函数的说明如下，

```go
Copyfunc find(num int, nums ...int)
```

由可变参数函数的定义可知，`nums ...int` 意味它可以接受 `int` 类型的可变参数。

在上面程序的第 23 行，`nums` 作为可变参数传入 `find` 函数。前面我们知道，这些可变参数参数会被转换为 `int` 类型切片然后在传入 `find` 函数中。但是在这里 `nums` 已经是一个 int 类型切片，编译器试图在 `nums` 基础上再创建一个切片，像下面这样

```go
Copyfind(89, []int{nums})
```

这里之所以会失败是因为 `nums` 是一个 `[]int`类型 而不是 `int`类型。

那么有没有办法给可变参数函数传入切片参数呢？答案是肯定的。

**有一个可以直接将切片传入可变参数函数的语法糖，你可以在在切片后加上 ... 后缀。如果这样做，切片将直接传入函数，不再创建新的切片**

在上面的程序中，如果你将第 23 行的 `find(89, nums)` 替换为 `find(89, nums...)` ，程序将成功编译并有如下输出

```go
Copytype of nums is []int
89 found at index 0 in [89 90 95]
```

下面是完整的程序供您参考。

```go
Copypackage main

import (
    "fmt"
)

func find(num int, nums ...int) {
    fmt.Printf("type of nums is %T\n", nums)
    found := false
    for i, v := range nums {
        if v == num {
            fmt.Println(num, "found at index", i, "in", nums)
            found = true
        }
    }
    if !found {
        fmt.Println(num, "not found in ", nums)
    }
    fmt.Printf("\n")
}
func main() {
    nums := []int{89, 90, 95}
    find(89, nums...)
}
```

## 不直观的错误

当你修改可变参数函数中的切片时，请确保你知道你正在做什么。

下面让我们来看一个简单的例子。

```go
Copypackage main

import (
    "fmt"
)

func change(s ...string) {  
    s[0] = "Go"
}

func main() {
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome)
}
```

你认为这段代码将输出什么呢？如果你认为它输出 `[Go world]` 。恭喜你！你已经理解了可变参数函数和切片。如果你猜错了，那也不要紧，让我来解释下为什么会有这样的输出。

在第 13 行，我们使用了语法糖 `...` 并且将切片作为可变参数传入 `change` 函数。

正如前面我们所讨论的，如果使用了 `...` ，`welcome` 切片本身会作为参数直接传入，不需要再创建一个新的切片。这样参数 `welcome` 将作为参数传入 `change` 函数

在 `change` 函数中，切片的第一个元素被替换成 `Go`，这样程序产生了下面的输出值

```
Copy[Go world]
```

这里还有一个例子来理解可变参数函数。

```go
Copypackage main

import (
    "fmt"
)

func change(s ...string) {
    s[0] = "Go"
    s = append(s, "playground")
    fmt.Println(s)
}

func main() {
    welcome := []string{"hello", "world"}
    change(welcome...)
    fmt.Println(welcome)
}
```

# 14. 字符串

## 什么是字符串？

Go 语言中的字符串是一个字节切片。把内容放在双引号""之间，我们可以创建一个字符串。让我们来看一个创建并打印字符串的简单示例。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    name := "Hello World"
    fmt.Println(name)
}
```

上面的程序将会输出 `Hello World`。

Go 中的字符串是兼容 Unicode 编码的，并且使用 UTF-8 进行编码。

## 单独获取字符串的每一个字节

由于字符串是一个字节切片，所以我们可以获取字符串的每一个字节。

```go
Copypackage main

import (
    "fmt"
)

func printBytes(s string) {
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}

func main() {
    name := "Hello World"
    printBytes(name)
}
```

上面程序的第 8 行，**len(s) 返回字符串中字节的数量**，然后我们用了一个 for 循环以 16 进制的形式打印这些字节。`%x` 格式限定符用于指定 16 进制编码。上面的程序输出 `48 65 6c 6c 6f 20 57 6f 72 6c 64`。这些打印出来的字符是 "Hello World" 以 [Unicode UTF-8 编码]的结果。为了更好的理解 go 中的字符串，需要对 Unicode 和 UTF-8 有基础的理解。我推荐阅读一下 https://naveenr.net/unicode-character-set-and-utf-8-utf-16-utf-32-encoding/ 来理解一下什么是 Unicode 和 UTF-8。

让我们稍微修改一下上面的程序，让它打印字符串的每一个字符。

```go
Copypackage main

import (
    "fmt"
)

func printBytes(s string) {
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}


func printChars(s string) {
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%c ",s[i])
    }
}

func main() {
    name := "Hello World"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
}
```

在 `printChars` 方法(第 16 行中)中，`%c` 格式限定符用于打印字符串的字符。这个程序输出结果是：

```
Copy48 65 6c 6c 6f 20 57 6f 72 6c 64  
H e l l o   W o r l d
```

上面的程序获取字符串的每一个字符，虽然看起来是合法的，但却有一个严重的 bug。让我拆解这个代码来看看我们做错了什么。

```go
Copypackage main

import (
    "fmt"
)

func printBytes(s string) {
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}

func printChars(s string) {
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%c ",s[i])
    }
}

func main() {
    name := "Hello World"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
    fmt.Printf("\n")
    name = "Señor"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
}
```

上面代码输出的结果是：

```
Copy48 65 6c 6c 6f 20 57 6f 72 6c 64  
H e l l o   W o r l d  
53 65 c3 b1 6f 72  
S e Ã ± o r
```

在上面程序的第 28 行，我们尝试输出 **Señor** 的字符，但却输出了错误的 **S e Ã ± o r**。 为什么程序分割 `Hello World` 时表现完美，但分割 `Señor` 就出现了错误呢？这是因为 `ñ` 的 Unicode 代码点（Code Point）是 `U+00F1`。它的 [UTF-8 编码]占用了 c3 和 b1 两个字节。它的 UTF-8 编码占用了两个字节 c3 和 b1。而我们打印字符时，却假定每个字符的编码只会占用一个字节，这是错误的。在 UTF-8 编码中，一个代码点可能会占用超过一个字节的空间。那么我们该怎么办呢？rune 能帮我们解决这个难题。

## rune

rune 是 Go 语言的内建类型，它也是 int32 的别称。在 Go 语言中，rune 表示一个代码点。代码点无论占用多少个字节，都可以用一个 rune 来表示。让我们修改一下上面的程序，用 rune 来打印字符。

```go
Copypackage main

import (
    "fmt"
)

func printBytes(s string) {
    for i:= 0; i < len(s); i++ {
        fmt.Printf("%x ", s[i])
    }
}

func printChars(s string) {
    runes := []rune(s)
    for i:= 0; i < len(runes); i++ {
        fmt.Printf("%c ",runes[i])
    }
}

func main() {
    name := "Hello World"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
    fmt.Printf("\n\n")
    name = "Señor"
    printBytes(name)
    fmt.Printf("\n")
    printChars(name)
}
```

在上面代码的第 14 行，字符串被转化为一个 rune 切片。然后我们循环打印字符。程序的输出结果是

```
Copy48 65 6c 6c 6f 20 57 6f 72 6c 64  
H e l l o   W o r l d 

53 65 c3 b1 6f 72  
S e ñ o r
```

上面的输出结果非常完美，就是我们想要的结果:)。

## 字符串的 for range 循环

上面的程序是一种遍历字符串的好方法，但是 Go 给我们提供了一种更简单的方法来做到这一点：使用 **for range** 循环。

```go
Copypackage main

import (
    "fmt"
)

func printCharsAndBytes(s string) {
    for index, rune := range s {
        fmt.Printf("%c starts at byte %d\n", rune, index)
    }
}

func main() {
    name := "Señor"
    printCharsAndBytes(name)
}
```

在上面程序中的第8行，使用 `for range` 循环遍历了字符串。循环返回的是是当前 rune 的字节位置。程序的输出结果为：

```
CopyS starts at byte 0  
e starts at byte 1  
ñ starts at byte 2
o starts at byte 4  
r starts at byte 5
```

从上面的输出中可以清晰的看到 `ñ` 占了两个字节:)。

## 用字节切片构造字符串

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    byteSlice := []byte{0x43, 0x61, 0x66, 0xC3, 0xA9}
    str := string(byteSlice)
    fmt.Println(str)
}
```

上面的程序中 `byteSlice` 包含字符串 `Café` 用 UTF-8 编码后的 16 进制字节。程序输出结果是 `Café`。

如果我们把 16 进制换成对应的 10 进制值会怎么样呢？上面的程序还能工作吗？让我们来试一试：

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    byteSlice := []byte{67, 97, 102, 195, 169}//decimal equivalent of {'\x43', '\x61', '\x66', '\xC3', '\xA9'}
    str := string(byteSlice)
    fmt.Println(str)
}
```

上面程序的输出结果也是`Café`

## 用 rune 切片构造字符串

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    runeSlice := []rune{0x0053, 0x0065, 0x00f1, 0x006f, 0x0072}
    str := string(runeSlice)
    fmt.Println(str)
}
```

在上面的程序中 `runeSlice` 包含字符串 `Señor`的 16 进制的 Unicode 代码点。这个程序将会输出`Señor`。

## 字符串的长度

[utf8 package] 包中的 `func RuneCountInString(s string) (n int)` 方法用来获取字符串的长度。这个方法传入一个字符串参数然后返回字符串中的 rune 的数量。

```go
Copypackage main

import (  
    "fmt"
    "unicode/utf8"
)

func length(s string) {  
    fmt.Printf("length of %s is %d\n", s, utf8.RuneCountInString(s))
}
func main() { 
    word1 := "Señor" 
    length(word1)
    word2 := "Pets"
    length(word2)
}
```

上面程序的输出结果是：

```
Copylength of Señor is 5  
length of Pets is 4
```

## 字符串是不可变的

Go 中的字符串是不可变的。一旦一个字符串被创建，那么它将无法被修改。

```go
Copypackage main

import (  
    "fmt"
)

func mutate(s string)string {  
    s[0] = 'a'//any valid unicode character within single quote is a rune 
    return s
}
func main() {  
    h := "hello"
    fmt.Println(mutate(h))
}
```

在上面程序中的第 8 行，我们试图把这个字符串中的第一个字符修改为 'a'。由于字符串是不可变的，因此这个操作是非法的。所以程序抛出了一个错误 **main.go:8: cannot assign to s[0]**。

为了修改字符串，可以把字符串转化为一个 rune 切片。然后这个切片可以进行任何想要的改变，然后再转化为一个字符串。

```go
Copypackage main

import (  
    "fmt"
)

func mutate(s []rune) string {  
    s[0] = 'a' 
    return string(s)
}
func main() {  
    h := "hello"
    fmt.Println(mutate([]rune(h)))
}
```

在上面程序的第 7 行，`mutate` 函数接收一个 rune 切片参数，它将切片的第一个元素修改为 `'a'`，然后将 rune 切片转化为字符串，并返回该字符串。程序的第 13 行调用了该函数。我们把 `h` 转化为一个 rune 切片，并传递给了 `mutate`。这个程序输出 `aello`。

# 15. 指针

### 什么是指针？

指针是一种存储变量内存地址（Memory Address）的变量。

[![img](https://img2018.cnblogs.com/blog/1825659/201910/1825659-20191023232851087-863913389.png)](https://img2018.cnblogs.com/blog/1825659/201910/1825659-20191023232851087-863913389.png)

如上图所示，变量 `b` 的值为 `156`，而 `b` 的内存地址为 `0x1040a124`。变量 `a` 存储了 `b` 的地址。我们就称 `a` 指向了 `b`。

### 指针的声明

指针变量的类型为 ***T**，该指针指向一个 **T** 类型的变量。

接下来我们写点代码。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    b := 255
    var a *int = &b
    fmt.Printf("Type of a is %T\n", a)
    fmt.Println("address of b is", a)
}
```

**&** 操作符用于获取变量的地址。上面程序的第 9 行我们把 `b` 的地址赋值给 ***int** 类型的 `a`。我们称 `a` 指向了 `b`。当我们打印 `a` 的值时，会打印出 `b` 的地址。程序将输出：

```
CopyType of a is *int  
address of b is 0x1040a124
```

由于 b 可能处于内存的任何位置，你应该会得到一个不同的地址。

### 指针的零值（Zero Value）

指针的零值是 `nil`。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    a := 25
    var b *int
    if b == nil {
        fmt.Println("b is", b)
        b = &a
        fmt.Println("b after initialization is", b)
    }
}
```

上面的程序中，`b` 初始化为 `nil`，接着将 `a` 的地址赋值给 `b`。程序会输出：

```
Copyb is <nil>  
b after initialisation is 0x1040a124
```

### 指针的解引用

指针的解引用可以获取指针所指向的变量的值。将 `a` 解引用的语法是 `*a`。

通过下面的代码，可以看到如何使用解引用。

```go
Copypackage main  
import (  
    "fmt"
)

func main() {  
    b := 255
    a := &b
    fmt.Println("address of b is", a)
    fmt.Println("value of b is", *a)
}
```

在上面程序的第 10 行，我们将 `a` 解引用，并打印了它的值。不出所料，我们会打印出 `b` 的值。程序会输出：

```
Copyaddress of b is 0x1040a124  
value of b is 255
```

我们再编写一个程序，用指针来修改 b 的值。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    b := 255
    a := &b
    fmt.Println("address of b is", a)
    fmt.Println("value of b is", *a)
    *a++
    fmt.Println("new value of b is", b)
}
```

在上面程序的第 12 行中，我们把 `a` 指向的值加 1，由于 `a` 指向了 `b`，因此 `b` 的值也发生了同样的改变。于是 `b` 的值变为 256。程序会输出：

```
Copyaddress of b is 0x1040a124  
value of b is 255  
new value of b is 256
```

### 向函数传递指针参数

```go
Copypackage main

import (  
    "fmt"
)

func change(val *int) {  
    *val = 55
}
func main() {  
    a := 58
    fmt.Println("value of a before function call is",a)
    b := &a
    change(b)
    fmt.Println("value of a after function call is", a)
}
```

在上面程序中的第 14 行，我们向函数 `change` 传递了指针变量 `b`，而 `b` 存储了 `a` 的地址。程序的第 8 行在 `change` 函数内使用解引用，修改了 a 的值。该程序会输出：

```
Copyvalue of a before function call is 58  
value of a after function call is 55
```

### 不要向函数传递数组的指针，而应该使用切片

假如我们想要在函数内修改一个数组，并希望调用函数的地方也能得到修改后的数组，一种解决方案是把一个指向数组的指针传递给这个函数。

```go
Copypackage main

import (  
    "fmt"
)

func modify(arr *[3]int) {  
    (*arr)[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```

在上面程序的第 13 行中，我们将数组的地址传递给了 `modify` 函数。在第 8 行，我们在 `modify` 函数里把 `arr` 解引用，并将 `90` 赋值给这个数组的第一个元素。程序会输出 `[90 90 91]`。

**a[x] 是 (\*a)[x] 的简写形式，因此上面代码中的 (\*arr)[0] 可以替换为 arr[0]**。下面我们用简写形式重写以上代码。

```go
Copypackage main

import (  
    "fmt"
)

func modify(arr *[3]int) {  
    arr[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```

该程序也会输出 `[90 90 91]`。

**这种方式向函数传递一个数组指针参数，并在函数内修改数组。尽管它是有效的，但却不是 Go 语言惯用的实现方式。我们最好使用切片来处理。**

接下来我们用[切片]来重写之前的代码。

```go
Copypackage main

import (  
    "fmt"
)

func modify(sls []int) {  
    sls[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(a[:])
    fmt.Println(a)
}
```

在上面程序的第 13 行，我们将一个切片传递给了 `modify` 函数。在 `modify` 函数中，我们把切片的第一个元素修改为 `90`。程序也会输出 `[90 90 91]`。**所以别再传递数组指针了，而是使用切片吧**。上面的代码更加简洁，也更符合 Go 语言的习惯。

### Go 不支持指针运算

Go 并不支持其他语言（例如 C）中的指针运算。

```go
Copypackage main

func main() {  
    b := [...]int{109, 110, 111}
    p := &b
    p++
}
```

上面的程序会抛出编译错误：**main.go:6: invalid operation: p++ (non-numeric type \*[3]int)**。16. 结构体

## 16.结构体

### 什么是结构体？

结构体是用户定义的类型，表示若干个字段（Field）的集合。有时应该把数据整合在一起，而不是让这些数据没有联系。这种情况下可以使用结构体。

例如，一个职员有 `firstName`、`lastName` 和 `age` 三个属性，而把这些属性组合在一个结构体 `employee` 中就很合理。

### 结构体的声明

```go
Copytype Employee struct {
    firstName string
    lastName  string
    age       int
}
```

在上面的代码片段里，声明了一个结构体类型 `Employee`，它有 `firstName`、`lastName` 和 `age` 三个字段。通过把相同类型的字段声明在同一行，结构体可以变得更加紧凑。在上面的结构体中，`firstName` 和 `lastName` 属于相同的 `string` 类型，于是这个结构体可以重写为：

```go
Copytype Employee struct {
    firstName, lastName string
    age, salary         int
}
```

上面的结构体 `Employee` 称为 **命名的结构体（Named Structure）**。我们创建了名为 `Employee` 的新类型，而它可以用于创建 `Employee` 类型的结构体变量。

声明结构体时也可以不用声明一个新类型，这样的结构体类型称为 **匿名结构体（Anonymous Structure）**。

```go
Copyvar employee struct {
    firstName, lastName string
    age int
}
```

上述代码片段创建一个**匿名结构体** `employee`。

### 创建命名的结构体

通过下面代码，我们定义了一个**命名的结构体 Employee**。

```go
Copypackage main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {

    //creating structure using field names
    emp1 := Employee{
        firstName: "Sam",
        age:       25,
        salary:    500,
        lastName:  "Anderson",
    }

    //creating structure without using field names
    emp2 := Employee{"Thomas", "Paul", 29, 800}

    fmt.Println("Employee 1", emp1)
    fmt.Println("Employee 2", emp2)
}
```

在上述程序的第 7 行，我们创建了一个命名的结构体 `Employee`。而在第 15 行，通过指定每个字段名的值，我们定义了结构体变量 `emp1`。字段名的顺序不一定要与声明结构体类型时的顺序相同。在这里，我们改变了 `lastName` 的位置，将其移到了末尾。这样做也不会有任何的问题。

在上面程序的第 23 行，定义 `emp2` 时我们省略了字段名。在这种情况下，就需要保证字段名的顺序与声明结构体时的顺序相同。

该程序将输出：

```
CopyEmployee 1 {Sam Anderson 25 500}
Employee 2 {Thomas Paul 29 800}
```

### 创建匿名结构体

```go
Copypackage main

import (
    "fmt"
)

func main() {
    emp3 := struct {
        firstName, lastName string
        age, salary         int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```

在上述程序的第 3 行，我们定义了一个**匿名结构体变量** `emp3`。上面我们已经提到，之所以称这种结构体是匿名的，是因为它只是创建一个新的结构体变量 `em3`，而没有定义任何结构体类型。

该程序会输出：

```
CopyEmployee 3 {Andreah Nikola 31 5000}
```

### 结构体的零值（Zero Value）

当定义好的结构体并没有被显式地初始化时，该结构体的字段将默认赋为零值。

```go
Copypackage main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp4 Employee //zero valued structure
    fmt.Println("Employee 4", emp4)
}
```

该程序定义了 `emp4`，却没有初始化任何值。因此 `firstName` 和 `lastName` 赋值为 string 的零值（`""`）。而 `age` 和 `salary` 赋值为 int 的零值（0）。该程序会输出：

```
CopyEmployee 4 { 0 0}
```

当然还可以为某些字段指定初始值，而忽略其他字段。这样，忽略的字段名会赋值为零值。

```go
Copypackage main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp5 := Employee{
        firstName: "John",
        lastName:  "Paul",
    }
    fmt.Println("Employee 5", emp5)
}
```

在上面程序中的第 14 行和第 15 行，我们初始化了 `firstName` 和 `lastName`，而 `age` 和 `salary` 没有进行初始化。因此 `age` 和 `salary` 赋值为零值。该程序会输出：

```
CopyEmployee 5 {John Paul 0 0}
```

### 访问结构体的字段

点号操作符 `.` 用于访问结构体的字段。

```go
Copypackage main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp6 := Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp6.firstName)
    fmt.Println("Last Name:", emp6.lastName)
    fmt.Println("Age:", emp6.age)
    fmt.Printf("Salary: $%d", emp6.salary)
}
```

上面程序中的 **emp6.firstName** 访问了结构体 `emp6` 的字段 `firstName`。该程序输出：

```
CopyFirst Name: Sam  
Last Name: Anderson  
Age: 55  
Salary: $6000
```

还可以创建零值的 `struct`，以后再给各个字段赋值。

```go
Copypackage main

import (
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    var emp7 Employee
    emp7.firstName = "Jack"
    emp7.lastName = "Adams"
    fmt.Println("Employee 7:", emp7)
}
```

在上面程序中，我们定义了 `emp7`，接着给 `firstName` 和 `lastName` 赋值。该程序会输出：

```
CopyEmployee 7: {Jack Adams 0 0}
```

### 结构体的指针

还可以创建指向结构体的指针。

```go
Copypackage main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", (*emp8).firstName)
    fmt.Println("Age:", (*emp8).age)
}
```

在上面程序中，**emp8** 是一个指向结构体 `Employee` 的指针。`(*emp8).firstName` 表示访问结构体 `emp8` 的 `firstName` 字段。该程序会输出：

```
CopyFirst Name: Sam
Age: 55
```

**Go 语言允许我们在访问 firstName 字段时，可以使用 emp8.firstName 来代替显式的解引用 (\*emp8).firstName**。

```go
Copypackage main

import (  
    "fmt"
)

type Employee struct {  
    firstName, lastName string
    age, salary         int
}

func main() {  
    emp8 := &Employee{"Sam", "Anderson", 55, 6000}
    fmt.Println("First Name:", emp8.firstName)
    fmt.Println("Age:", emp8.age)
}
```

在上面的程序中，我们使用 `emp8.firstName` 来访问 `firstName` 字段，该程序会输出：

```
CopyFirst Name: Sam
Age: 55
```

### 匿名字段

当我们创建结构体时，字段可以只有类型，而没有字段名。这样的字段称为匿名字段（Anonymous Field）。

以下代码创建一个 `Person` 结构体，它含有两个匿名字段 `string` 和 `int`。

```go
Copytype Person struct {  
    string
    int
}
```

我们接下来使用匿名字段来编写一个程序。

```go
Copypackage main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    p := Person{"Naveen", 50}
    fmt.Println(p)
}
```

在上面的程序中，结构体 `Person` 有两个匿名字段。`p := Person{"Naveen", 50}` 定义了一个 `Person` 类型的变量。该程序输出 `{Naveen 50}`。

**虽然匿名字段没有名称，但其实匿名字段的名称就默认为它的类型**。比如在上面的 `Person` 结构体里，虽说字段是匿名的，但 Go 默认这些字段名是它们各自的类型。所以 `Person` 结构体有两个名为 `string` 和 `int` 的字段。

```go
Copypackage main

import (  
    "fmt"
)

type Person struct {  
    string
    int
}

func main() {  
    var p1 Person
    p1.string = "naveen"
    p1.int = 50
    fmt.Println(p1)
}
```

在上面程序的第 14 行和第 15 行，我们访问了 `Person` 结构体的匿名字段，我们把字段类型作为字段名，分别为 "string" 和 "int"。上面程序的输出如下：

```
Copy{naveen 50}
```

### 嵌套结构体（Nested Structs）

结构体的字段有可能也是一个结构体。这样的结构体称为嵌套结构体。

```go
Copypackage main

import (  
    "fmt"
)

type Address struct {  
    city, state string
}
type Person struct {  
    name string
    age int
    address Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.address = Address {
        city: "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:",p.age)
    fmt.Println("City:",p.address.city)
    fmt.Println("State:",p.address.state)
}
```

上面的结构体 `Person` 有一个字段 `address`，而 `address` 也是结构体。该程序输出：

```
CopyName: Naveen  
Age: 50  
City: Chicago  
State: Illinois
```

### 提升字段（Promoted Fields）

如果是结构体中有匿名的结构体类型字段，则该匿名结构体里的字段就称为提升字段。这是因为提升字段就像是属于外部结构体一样，可以用外部结构体直接访问。我知道这种定义很复杂，所以我们直接研究下代码来理解吧。

```go
Copytype Address struct {  
    city, state string
}
type Person struct {  
    name string
    age  int
    Address
}
```

在上面的代码片段中，`Person` 结构体有一个匿名字段 `Address`，而 `Address` 是一个结构体。现在结构体 `Address` 有 `city` 和 `state` 两个字段，访问这两个字段就像在 `Person` 里直接声明的一样，因此我们称之为提升字段。

```go
Copypackage main

import (
    "fmt"
)

type Address struct {
    city, state string
}
type Person struct {
    name string
    age  int
    Address
}

func main() {  
    var p Person
    p.name = "Naveen"
    p.age = 50
    p.Address = Address{
        city:  "Chicago",
        state: "Illinois",
    }
    fmt.Println("Name:", p.name)
    fmt.Println("Age:", p.age)
    fmt.Println("City:", p.city) //city is promoted field
    fmt.Println("State:", p.state) //state is promoted field
}
```

在上面代码中的第 26 行和第 27 行，我们使用了语法 `p.city` 和 `p.state`，访问提升字段 `city` 和 `state` 就像它们是在结构体 `p` 中声明的一样。该程序会输出：

```
CopyName: Naveen  
Age: 50  
City: Chicago  
State: Illinois
```

### 导出结构体和字段

如果结构体名称以大写字母开头，则它是其他包可以访问的导出类型（Exported Type）。同样，如果结构体里的字段首字母大写，它也能被其他包访问到。

让我们使用自定义包，编写一个程序来更好地去理解它。

在你的 Go 工作区的 `src` 目录中，创建一个名为 `structs` 的文件夹。另外在 `structs` 中再创建一个目录 `computer`。

在 `computer` 目录中，在名为 `spec.go` 的文件中保存下面的程序。

```go
Copypackage computer

type Spec struct { //exported struct  
    Maker string //exported field
    model string //unexported field
    Price int //exported field
}
```

上面的代码片段中，创建了一个 `computer` 包，里面有一个导出结构体类型 `Spec`。`Spec` 有两个导出字段 `Maker` 和 `Price`，和一个未导出的字段 `model`。接下来我们会在 main 包中导入这个包，并使用 `Spec` 结构体。

```go
Copypackage main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    fmt.Println("Spec:", spec)
}
```

包结构如下所示：

```
Copysrc  
   structs
        computer
            spec.go
        main.go
```

在上述程序的第 3 行，我们导入了 `computer` 包。在第 8 行和第 9 行，我们访问了结构体 `Spec` 的两个导出字段 `Maker` 和 `Price`。执行命令 `go install structs` 和 `workspacepath/bin/structs`，运行该程序。

如果我们试图访问未导出的字段 `model`，编译器会报错。将 `main.go` 的内容替换为下面的代码。

```go
Copypackage main

import "structs/computer"  
import "fmt"

func main() {  
    var spec computer.Spec
    spec.Maker = "apple"
    spec.Price = 50000
    spec.model = "Mac Mini"
    fmt.Println("Spec:", spec)
}
```

在上面程序的第 10 行，我们试图访问未导出的字段 `model`。如果运行这个程序，编译器会产生错误：**spec.model undefined (cannot refer to unexported field or method model)**。

### 结构体相等性（Structs Equality）

**结构体是值类型。如果它的每一个字段都是可比较的，则该结构体也是可比较的。如果两个结构体变量的对应字段相等，则这两个变量也是相等的**。

```go
Copypackage main

import (  
    "fmt"
)

type name struct {  
    firstName string
    lastName string
}


func main() {  
    name1 := name{"Steve", "Jobs"}
    name2 := name{"Steve", "Jobs"}
    if name1 == name2 {
        fmt.Println("name1 and name2 are equal")
    } else {
        fmt.Println("name1 and name2 are not equal")
    }

    name3 := name{firstName:"Steve", lastName:"Jobs"}
    name4 := name{}
    name4.firstName = "Steve"
    if name3 == name4 {
        fmt.Println("name3 and name4 are equal")
    } else {
        fmt.Println("name3 and name4 are not equal")
    }
}
```

在上面的代码中，结构体类型 `name` 包含两个 `string` 类型。由于字符串是可比较的，因此可以比较两个 `name` 类型的结构体变量。

上面代码中 `name1` 和 `name2` 相等，而 `name3` 和 `name4` 不相等。该程序会输出：

```
Copyname1 and name2 are equal  
name3 and name4 are not equal
```

**如果结构体包含不可比较的字段，则结构体变量也不可比较。**

```go
Copypackage main

import (  
    "fmt"
)

type image struct {  
    data map[int]int
}

func main() {  
    image1 := image{data: map[int]int{
        0: 155,
    }}
    image2 := image{data: map[int]int{
        0: 155,
    }}
    if image1 == image2 {
        fmt.Println("image1 and image2 are equal")
    }
}
```

在上面代码中，结构体类型 `image` 包含一个 `map` 类型的字段。由于 `map` 类型是不可比较的，因此 `image1` 和 `image2` 也不可比较。如果运行该程序，编译器会报错：**main.go:18: invalid operation: image1 == image2 (struct containing map[int]int cannot be compared)**。17. 方法

## 17.方法

### 什么是方法？

方法其实就是一个函数，在 `func` 这个关键字和方法名中间加入了一个特殊的接收器类型。接收器可以是结构体类型或者是非结构体类型。接收器是可以在方法的内部访问的。

下面就是创建一个方法的语法。

```go
Copyfunc (t Type) methodName(parameter list) {
}
```

上面的代码片段创建了一个接收器类型为 `Type` 的方法 `methodName`。

### 方法示例

让我们来编写一个简单的小程序，它会在结构体类型上创建一个方法并调用它。

```go
Copypackage main

import (
    "fmt"
)

type Employee struct {
    name     string
    salary   int
    currency string
}

/*
  displaySalary() 方法将 Employee 做为接收器类型
*/
func (e Employee) displaySalary() {
    fmt.Printf("Salary of %s is %s%d", e.name, e.currency, e.salary)
}

func main() {
    emp1 := Employee {
        name:     "Sam Adolf",
        salary:   5000,
        currency: "$",
    }
    emp1.displaySalary() // 调用 Employee 类型的 displaySalary() 方法
}
```

在上面程序的第 16 行，我们在 `Employee` 结构体类型上创建了一个 `displaySalary` 方法。displaySalary()方法在方法的内部访问了接收器 `e Employee`。在第 17 行，我们使用接收器 `e`，并打印 employee 的 name、currency 和 salary 这 3 个字段。

在第 26 行，我们调用了方法 `emp1.displaySalary()`。

程序输出：`Salary of Sam Adolf is $5000`。

### 为什么我们已经有函数了还需要方法呢？

上面的程序已经被重写为只使用函数，没有方法。

```go
Copypackage main

import (
    "fmt"
)

type Employee struct {
    name     string
    salary   int
    currency string
}

/*
displaySalary()方法被转化为一个函数，把 Employee 当做参数传入。
*/
func displaySalary(e Employee) {
    fmt.Printf("Salary of %s is %s%d", e.name, e.currency, e.salary)
}

func main() {
    emp1 := Employee{
        name:     "Sam Adolf",
        salary:   5000,
        currency: "$",
    }
    displaySalary(emp1)
}
```

在上面的程序中，`displaySalary` 方法被转化为一个函数，`Employee` 结构体被当做参数传递给它。这个程序也产生完全相同的输出：`Salary of Sam Adolf is $5000`。

既然我们可以使用函数写出相同的程序，那么为什么我们需要方法？这有着几个原因，让我们一个个的看看。

- `Go 不是纯粹的面向对象编程语言`，而且Go不支持类。因此，基于类型的方法是一种实现和类相似行为的途径。
- 相同的名字的方法可以定义在不同的类型上，而相同名字的函数是不被允许的。假设我们有一个 `Square` 和 `Circle` 结构体。可以在 `Square` 和 `Circle` 上分别定义一个 `Area` 方法。见下面的程序。

```go
Copypackage main

import (
    "fmt"
    "math"
)

type Rectangle struct {
    length int
    width  int
}

type Circle struct {
    radius float64
}

func (r Rectangle) Area() int {
    return r.length * r.width
}

func (c Circle) Area() float64 {
    return math.Pi * c.radius * c.radius
}

func main() {
    r := Rectangle{
        length: 10,
        width:  5,
    }
    fmt.Printf("Area of rectangle %d\n", r.Area())
    c := Circle{
        radius: 12,
    }
    fmt.Printf("Area of circle %f", c.Area())
}
```

该程序输出：

```
CopyArea of rectangle 50
Area of circle 452.389342
```

上面方法的属性被使用在接口中。我们将在接下来的教程中讨论这个问题。

### 指针接收器与值接收器

到目前为止，我们只看到了使用值接收器的方法。还可以创建使用指针接收器的方法。值接收器和指针接收器之间的区别在于，在指针接收器的方法内部的改变对于调用者是可见的，然而值接收器的情况不是这样的。让我们用下面的程序来帮助理解这一点。

```go
Copypackage main

import (
    "fmt"
)

type Employee struct {
    name string
    age  int
}

/*
使用值接收器的方法。
*/
func (e Employee) changeName(newName string) {
    e.name = newName
}

/*
使用指针接收器的方法。
*/
func (e *Employee) changeAge(newAge int) {
    e.age = newAge
}

func main() {
    e := Employee{
        name: "Mark Andrew",
        age:  50,
    }
    fmt.Printf("Employee name before change: %s", e.name)
    e.changeName("Michael Andrew")
    fmt.Printf("\nEmployee name after change: %s", e.name)

    fmt.Printf("\n\nEmployee age before change: %d", e.age)
    (&e).changeAge(51)
    fmt.Printf("\nEmployee age after change: %d", e.age)
}
```

在上面的程序中，`changeName` 方法有一个值接收器 `(e Employee)`，而 `changeAge` 方法有一个指针接收器 `(e *Employee)`。在 `changeName` 方法中对 `Employee` 结构体的字段 `name` 所做的改变对调用者是不可见的，因此程序在调用 `e.changeName("Michael Andrew")` 这个方法的前后打印出相同的名字。由于 `changeAge` 方法是使用指针 `(e *Employee)` 接收器的，所以在调用 `(&e).changeAge(51)` 方法对 `age` 字段做出的改变对调用者将是可见的。该程序输出如下：

```
CopyEmployee name before change: Mark Andrew
Employee name after change: Mark Andrew

Employee age before change: 50
Employee age after change: 51
```

在上面程序的第 36 行，我们使用 `(&e).changeAge(51)` 来调用 `changeAge` 方法。由于 `changeAge` 方法有一个指针接收器，所以我们使用 `(&e)` 来调用这个方法。其实没有这个必要，Go语言让我们可以直接使用 `e.changeAge(51)`。`e.changeAge(51)` 会自动被Go语言解释为 `(&e).changeAge(51)`。

下面的程序重写了，使用 `e.changeAge(51)` 来代替 `(&e).changeAge(51)`，它输出相同的结果。

```go
Copypackage main

import (
    "fmt"
)

type Employee struct {
    name string
    age  int
}

/*
使用值接收器的方法。
*/
func (e Employee) changeName(newName string) {
    e.name = newName
}

/*
使用指针接收器的方法。
*/
func (e *Employee) changeAge(newAge int) {
    e.age = newAge
}

func main() {
    e := Employee{
        name: "Mark Andrew",
        age:  50,
    }
    fmt.Printf("Employee name before change: %s", e.name)
    e.changeName("Michael Andrew")
    fmt.Printf("\nEmployee name after change: %s", e.name)

    fmt.Printf("\n\nEmployee age before change: %d", e.age)
    e.changeAge(51)
    fmt.Printf("\nEmployee age after change: %d", e.age)
}
```

### 那么什么时候使用指针接收器，什么时候使用值接收器？

一般来说，指针接收器可以使用在：对方法内部的接收器所做的改变应该对调用者可见时。

指针接收器也可以被使用在如下场景：当拷贝一个结构体的代价过于昂贵时。考虑下一个结构体有很多的字段。在方法内使用这个结构体做为值接收器需要拷贝整个结构体，这是很昂贵的。在这种情况下使用指针接收器，结构体不会被拷贝，只会传递一个指针到方法内部使用。

在其他的所有情况，值接收器都可以被使用。

### 匿名字段的方法

属于结构体的匿名字段的方法可以被直接调用，就好像这些方法是属于定义了匿名字段的结构体一样。

```go
Copypackage main

import (
    "fmt"
)

type address struct {
    city  string
    state string
}

func (a address) fullAddress() {
    fmt.Printf("Full address: %s, %s", a.city, a.state)
}

type person struct {
    firstName string
    lastName  string
    address
}

func main() {
    p := person{
        firstName: "Elon",
        lastName:  "Musk",
        address: address {
            city:  "Los Angeles",
            state: "California",
        },
    }

    p.fullAddress() //访问 address 结构体的 fullAddress 方法
}
```

在上面程序的第 32 行，我们通过使用 `p.fullAddress()` 来访问 `address` 结构体的 `fullAddress()` 方法。明确的调用 `p.address.fullAddress()` 是没有必要的。该程序输出：

```
CopyFull address: Los Angeles, California
```

### 在方法中使用值接收器 与 在函数中使用值参数

这个话题很多Go语言新手都弄不明白。我会尽量讲清楚。

当一个函数有一个值参数，它只能接受一个值参数。

当一个方法有一个值接收器，它可以接受值接收器和指针接收器。

让我们通过一个例子来理解这一点。

```go
Copypackage main

import (
    "fmt"
)

type rectangle struct {
    length int
    width  int
}

func area(r rectangle) {
    fmt.Printf("Area Function result: %d\n", (r.length * r.width))
}

func (r rectangle) area() {
    fmt.Printf("Area Method result: %d\n", (r.length * r.width))
}

func main() {
    r := rectangle{
        length: 10,
        width:  5,
    }
    area(r)
    r.area()

    p := &r
    /*
       compilation error, cannot use p (type *rectangle) as type rectangle
       in argument to area
    */
    //area(p)

    p.area()//通过指针调用值接收器
}
```

第 12 行的函数 `func area(r rectangle)` 接受一个值参数，方法 `func (r rectangle) area()` 接受一个值接收器。

在第 25 行，我们通过值参数 `area(r)` 来调用 area 这个函数，这是合法的。同样，我们使用值接收器来调用 area 方法 `r.area()`，这也是合法的。

在第 28 行，我们创建了一个指向 `r` 的指针 `p`。如果我们试图把这个指针传递到只能接受一个值参数的函数 area，编译器将会报错。所以我把代码的第 33 行注释了。如果你把这行的代码注释去掉，编译器将会抛出错误 `compilation error, cannot use p (type *rectangle) as type rectangle in argument to area.`。这将会按预期抛出错误。

现在到了棘手的部分了，在第35行的代码 `p.area()` 使用指针接收器 `p` 调用了只接受一个值接收器的方法 `area`。这是完全有效的。原因是当 `area` 有一个值接收器时，为了方便Go语言把 `p.area()` 解释为 `(*p).area()`。

该程序将会输出：

```
CopyArea Function result: 50
Area Method result: 50
Area Method result: 50
```

### 在方法中使用指针接收器 与 在函数中使用指针参数

和值参数相类似，函数使用指针参数只接受指针，而使用指针接收器的方法可以使用值接收器和指针接收器。

```go
Copypackage main

import (
    "fmt"
)

type rectangle struct {
    length int
    width  int
}

func perimeter(r *rectangle) {
    fmt.Println("perimeter function output:", 2*(r.length+r.width))

}

func (r *rectangle) perimeter() {
    fmt.Println("perimeter method output:", 2*(r.length+r.width))
}

func main() {
    r := rectangle{
        length: 10,
        width:  5,
    }
    p := &r //pointer to r
    perimeter(p)
    p.perimeter()

    /*
        cannot use r (type rectangle) as type *rectangle in argument to perimeter
    */
    //perimeter(r)

    r.perimeter()//使用值来调用指针接收器
}
```

在上面程序的第 12 行，定义了一个接受指针参数的函数 `perimeter`。第 17 行定义了一个有一个指针接收器的方法。

在第 27 行，我们调用 perimeter 函数时传入了一个指针参数。在第 28 行，我们通过指针接收器调用了 perimeter 方法。所有一切看起来都这么完美。

在被注释掉的第 33 行，我们尝试通过传入值参数 `r` 调用函数 `perimeter`。这是不被允许的，因为函数的指针参数不接受值参数。如果你把这行的代码注释去掉并把程序运行起来，编译器将会抛出错误 `main.go:33: cannot use r (type rectangle) as type *rectangle in argument to perimeter.`。

在第 35 行，我们通过值接收器 `r` 来调用有指针接收器的方法 `perimeter`。这是被允许的，为了方便Go语言把代码 `r.perimeter()` 解释为 `(&r).perimeter()`。该程序输出：

```
Copyperimeter function output: 30
perimeter method output: 30
perimeter method output: 30
```

### 在非结构体上的方法

到目前为止，我们只在结构体类型上定义方法。也可以在非结构体类型上定义方法，但是有一个问题。**为了在一个类型上定义一个方法，方法的接收器类型定义和方法的定义应该在同一个包中。到目前为止，我们定义的所有结构体和结构体上的方法都是在同一个 main 包中，因此它们是可以运行的。**

```go
Copypackage main

func (a int) add(b int) {
}

func main() {

}
```

在上面程序的第 3 行，我们尝试把一个 `add` 方法添加到内置的类型 `int`。这是不允许的，因为 `add` 方法的定义和 `int` 类型的定义不在同一个包中。该程序会抛出编译错误 `cannot define new methods on non-local type int`。

让该程序工作的方法是为内置类型 int 创建一个类型别名，然后创建一个以该类型别名为接收器的方法。

```go
Copypackage main

import "fmt"

type myInt int

func (a myInt) add(b myInt) myInt {
    return a + b
}

func main() {
    num1 := myInt(5)
    num2 := myInt(10)
    sum := num1.add(num2)
    fmt.Println("Sum is", sum)
}
```

在上面程序的第5行，我们为 `int` 创建了一个类型别名 `myInt`。在第7行，我们定义了一个以 `myInt` 为接收器的的方法 `add`。

该程序将会打印出 `Sum is 15`。

我已经创建了一个程序，包含了我们迄今为止所讨论的所有概念

## 18.接口(一)

### 什么是接口？

在面向对象的领域里，接口一般这样定义：**接口定义一个对象的行为**。接口只指定了对象应该做什么，至于如何实现这个行为（即实现细节），则由对象本身去确定。

在 Go 语言中，接口就是方法签名（Method Signature）的集合。当一个类型定义了接口中的所有方法，我们称它实现了该接口。这与面向对象编程（OOP）的说法很类似。**接口指定了一个类型应该具有的方法，并由该类型决定如何实现这些方法**。

例如，`WashingMachine` 是一个含有 `Cleaning()` 和 `Drying()` 两个方法的接口。任何定义了 `Cleaning()` 和 `Drying()` 的类型，都称它实现了 `WashingMachine` 接口。

### 接口的声明与实现

让我们编写代码，创建一个接口并且实现它。

```go
Copypackage main

import (  
    "fmt"
)

//interface definition
type VowelsFinder interface {  
    FindVowels() []rune
}

type MyString string

//MyString implements VowelsFinder
func (ms MyString) FindVowels() []rune {  
    var vowels []rune
    for _, rune := range ms {
        if rune == 'a' || rune == 'e' || rune == 'i' || rune == 'o' || rune == 'u' {
            vowels = append(vowels, rune)
        }
    }
    return vowels
}

func main() {  
    name := MyString("Sam Anderson")
    var v VowelsFinder
    v = name // possible since MyString implements VowelsFinder
    fmt.Printf("Vowels are %c", v.FindVowels())

}
```

在上面程序的第 8 行，创建了一个名为 `VowelsFinder` 的接口，该接口有一个 `FindVowels() []rune` 的方法。

在接下来的一行，我们创建了一个 `MyString` 类型。

**在第 15 行，我们给接受者类型（Receiver Type） MyString 添加了方法 FindVowels() []rune。现在，我们称 MyString 实现了 VowelsFinder 接口。这就和其他语言（如 Java）很不同，其他一些语言要求一个类使用 implement 关键字，来显式地声明该类实现了接口。而在 Go 中，并不需要这样。如果一个类型包含了接口中声明的所有方法，那么它就隐式地实现了 Go 接口**。

在第 28 行，`v` 的类型为 `VowelsFinder`，`name` 的类型为 `MyString`，我们把 `name` 赋值给了 `v`。由于 `MyString` 实现了 `VowelFinder`，因此这是合法的。在下一行，`v.FindVowels()` 调用了 `MyString` 类型的 `FindVowels` 方法，打印字符串 `Sam Anderson` 里所有的元音。该程序输出 `Vowels are [a e o]`。

祝贺！你已经创建并实现了你的第一个接口。

### 接口的实际用途

前面的例子教我们创建并实现了接口，但还没有告诉我们接口的实际用途。在上面的程序里，如果我们使用 `name.FindVowels()`，而不是 `v.FindVowels()`，程序依然能够照常运行，但接口并没有体现出实际价值。

因此，我们现在讨论一下接口的实际应用场景。

我们编写一个简单程序，根据公司员工的个人薪资，计算公司的总支出。为了简单起见，我们假定支出的单位都是美元。

```go
Copypackage main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    CalculateSalary() int
}

type Permanent struct {  
    empId    int
    basicpay int
    pf       int
}

type Contract struct {  
    empId  int
    basicpay int
}

//salary of permanent employee is sum of basic pay and pf
func (p Permanent) CalculateSalary() int {  
    return p.basicpay + p.pf
}

//salary of contract employee is the basic pay alone
func (c Contract) CalculateSalary() int {  
    return c.basicpay
}

/*
total expense is calculated by iterating though the SalaryCalculator slice and summing  
the salaries of the individual employees  
*/
func totalExpense(s []SalaryCalculator) {  
    expense := 0
    for _, v := range s {
        expense = expense + v.CalculateSalary()
    }
    fmt.Printf("Total Expense Per Month $%d", expense)
}

func main() {  
    pemp1 := Permanent{1, 5000, 20}
    pemp2 := Permanent{2, 6000, 30}
    cemp1 := Contract{3, 3000}
    employees := []SalaryCalculator{pemp1, pemp2, cemp1}
    totalExpense(employees)

}
```

上面程序的第 7 行声明了一个 `SalaryCalculator` 接口类型，它只有一个方法 `CalculateSalary() int`。

在公司里，我们有两类员工，即第 11 行和第 17 行定义的结构体：`Permanent` 和 `Contract`。长期员工（`Permanent`）的薪资是 `basicpay` 与 `pf` 相加之和，而合同员工（`Contract`）只有基本工资 `basicpay`。在第 23 行和第 28 行中，方法 `CalculateSalary` 分别实现了以上关系。由于 `Permanent` 和 `Contract` 都声明了该方法，因此它们都实现了 `SalaryCalculator` 接口。

第 36 行声明的 `totalExpense` 方法体现出了接口的妙用。该方法接收一个 `SalaryCalculator` 接口的切片（`[]SalaryCalculator`）作为参数。在第 49 行，我们向 `totalExpense` 方法传递了一个包含 `Permanent` 和 `Contact` 类型的切片。在第 39 行中，通过调用不同类型对应的 `CalculateSalary` 方法，`totalExpense` 可以计算得到支出。

这样做最大的优点是：`totalExpense` 可以扩展新的员工类型，而不需要修改任何代码。假如公司增加了一种新的员工类型 `Freelancer`，它有着不同的薪资结构。`Freelancer`只需传递到 `totalExpense` 的切片参数中，无需 `totalExpense` 方法本身进行修改。只要 `Freelancer` 也实现了 `SalaryCalculator` 接口，`totalExpense` 就能够实现其功能。

该程序输出 `Total Expense Per Month $14050`。

### 接口的内部表示

我们可以把接口看作内部的一个元组 `(type, value)`。 `type` 是接口底层的具体类型（Concrete Type），而 `value` 是具体类型的值。

我们编写一个程序来更好地理解它。

```go
Copypackage main

import (  
    "fmt"
)

type Test interface {  
    Tester()
}

type MyFloat float64

func (m MyFloat) Tester() {  
    fmt.Println(m)
}

func describe(t Test) {  
    fmt.Printf("Interface type %T value %v\n", t, t)
}

func main() {  
    var t Test
    f := MyFloat(89.7)
    t = f
    describe(t)
    t.Tester()
}
```

`Test` 接口只有一个方法 `Tester()`，而 `MyFloat` 类型实现了该接口。在第 24 行，我们把变量 `f`（`MyFloat` 类型）赋值给了 `t`（`Test` 类型）。现在 `t` 的具体类型为 `MyFloat`，而 `t` 的值为 `89.7`。第 17 行的 `describe` 函数打印出了接口的具体类型和值。该程序输出：

```
CopyInterface type main.MyFloat value 89.7  
89.7
```

### 空接口

没有包含方法的接口称为空接口。空接口表示为 `interface{}`。由于空接口没有方法，因此所有类型都实现了空接口。

```go
Copypackage main

import (  
    "fmt"
)

func describe(i interface{}) {  
    fmt.Printf("Type = %T, value = %v\n", i, i)
}

func main() {  
    s := "Hello World"
    describe(s)
    i := 55
    describe(i)
    strt := struct {
        name string
    }{
        name: "Naveen R",
    }
    describe(strt)
}
```

在上面的程序的第 7 行，`describe(i interface{})` 函数接收空接口作为参数，因此，可以给这个函数传递任何类型。

在第 13 行、第 15 行和第 21 行，我们分别给 `describe` 函数传递了 `string`、`int` 和 `struct`。该程序打印：

```
CopyType = string, value = Hello World  
Type = int, value = 55  
Type = struct { name string }, value = {Naveen R}
```

### 类型断言

类型断言用于提取接口的底层值（Underlying Value）。

在语法 `i.(T)` 中，接口 `i` 的具体类型是 `T`，该语法用于获得接口的底层值。

一段代码胜过千言。下面编写个关于类型断言的程序。

```go
Copypackage main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) //get the underlying int value from i
    fmt.Println(s)
}
func main() {  
    var s interface{} = 56
    assert(s)
}
```

在第 12 行，`s` 的具体类型是 `int`。在第 8 行，我们使用了语法 `i.(int)` 来提取 `i` 的底层 int 值。该程序会打印 `56`。

在上面程序中，如果具体类型不是 int，会发生什么呢？接下来看看。

```go
Copypackage main

import (  
    "fmt"
)

func assert(i interface{}) {  
    s := i.(int) 
    fmt.Println(s)
}
func main() {  
    var s interface{} = "Steven Paul"
    assert(s)
}
```

在上面程序中，我们把具体类型为 `string` 的 `s` 传递给了 `assert` 函数，试图从它提取出 int 值。该程序会报错：`panic: interface conversion: interface {} is string, not int.`。

要解决该问题，我们可以使用以下语法：

```go
Copyv, ok := i.(T)
```

如果 `i` 的具体类型是 `T`，那么 `v` 赋值为 `i` 的底层值，而 `ok` 赋值为 `true`。

如果 `i` 的具体类型不是 `T`，那么 `ok` 赋值为 `false`，`v` 赋值为 `T` 类型的零值，**此时程序不会报错**。

```go
Copypackage main

import (  
    "fmt"
)

func assert(i interface{}) {  
    v, ok := i.(int)
    fmt.Println(v, ok)
}
func main() {  
    var s interface{} = 56
    assert(s)
    var i interface{} = "Steven Paul"
    assert(i)
}
```

当给 `assert` 函数传递 `Steven Paul` 时，由于 `i` 的具体类型不是 `int`，`ok` 赋值为 `false`，而 `v` 赋值为 0（int 的零值）。该程序打印：

```
Copy56 true  
0 false
```

### 类型选择（Type Switch）

类型选择用于将接口的具体类型与很多 case 语句所指定的类型进行比较。它与一般的 switch 语句类似。唯一的区别在于类型选择指定的是类型，而一般的 switch 指定的是值。

类型选择的语法类似于类型断言。类型断言的语法是 `i.(T)`，而对于类型选择，类型 `T` 由关键字 `type` 代替。下面看看程序是如何工作的。

```go
Copypackage main

import (  
    "fmt"
)

func findType(i interface{}) {  
    switch i.(type) {
    case string:
        fmt.Printf("I am a string and my value is %s\n", i.(string))
    case int:
        fmt.Printf("I am an int and my value is %d\n", i.(int))
    default:
        fmt.Printf("Unknown type\n")
    }
}
func main() {  
    findType("Naveen")
    findType(77)
    findType(89.98)
}
```

在上述程序的第 8 行，`switch i.(type)` 表示一个类型选择。每个 case 语句都把 `i` 的具体类型和一个指定类型进行了比较。如果 case 匹配成功，会打印出相应的语句。该程序输出：

```
CopyI am a string and my value is Naveen  
I am an int and my value is 77  
Unknown type
```

第 20 行中的 `89.98` 的类型是 `float64`，没有在 case 上匹配成功，因此最后一行打印了 `Unknown type`。

**还可以将一个类型和接口相比较。如果一个类型实现了接口，那么该类型与其实现的接口就可以互相比较**。

为了阐明这一点，下面写一个程序。

```go
Copypackage main

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

在上面程序中，结构体 `Person` 实现了 `Describer` 接口。在第 19 行的 case 语句中，`v` 与接口类型 `Describer` 进行了比较。`p` 实现了 `Describer`，因此满足了该 case 语句，于是当程序运行到第 32 行的 `findType(p)` 时，程序调用了 `Describe()` 方法。

该程序输出：

```
Copyunknown type  
Naveen R is 25 years old
```

接口（一）的内容到此结束。在接口（二）中我们还会继续讨论接口

# 19. 接口（二）

### 实现接口：指针接受者与值接受者

在接口（一）上的所有示例中，我们都是使用值接受者（Value Receiver）来实现接口的。我们同样可以使用指针接受者（Pointer Receiver）来实现接口。只不过在用指针接受者实现接口时，还有一些细节需要注意。我们通过下面的代码来理解吧。

```go
Copypackage main

import "fmt"

type Describer interface {  
    Describe()
}
type Person struct {  
    name string
    age  int
}

func (p Person) Describe() { // 使用值接受者实现  
    fmt.Printf("%s is %d years old\n", p.name, p.age)
}

type Address struct {
    state   string
    country string
}

func (a *Address) Describe() { // 使用指针接受者实现
    fmt.Printf("State %s Country %s", a.state, a.country)
}

func main() {  
    var d1 Describer
    p1 := Person{"Sam", 25}
    d1 = p1
    d1.Describe()
    p2 := Person{"James", 32}
    d1 = &p2
    d1.Describe()

    var d2 Describer
    a := Address{"Washington", "USA"}

    /* 如果下面一行取消注释会导致编译错误：
       cannot use a (type Address) as type Describer
       in assignment: Address does not implement
       Describer (Describe method has pointer
       receiver)
    */
    //d2 = a

    d2 = &a // 这是合法的
    // 因为在第 22 行，Address 类型的指针实现了 Describer 接口
    d2.Describe()

}
```

在上面程序中的第 13 行，结构体 `Person` 使用值接受者，实现了 `Describer` 接口。

我们在讨论方法的时候就已经提到过，使用值接受者声明的方法，既可以用值来调用，也能用指针调用。**不管是一个值，还是一个可以解引用的指针，调用这样的方法都是合法的**。

`p1` 的类型是 `Person`，在第 29 行，`p1` 赋值给了 `d1`。由于 `Person` 实现了接口变量 `d1`，因此在第 30 行，会打印 `Sam is 25 years old`。

接下来在第 32 行，`d1` 又赋值为 `&p2`，在第 33 行同样打印输出了 `James is 32 years old`。棒棒哒。😃

在 22 行，结构体 `Address` 使用指针接受者实现了 `Describer` 接口。

在上面程序里，如果去掉第 45 行的注释，我们会得到编译错误：`main.go:42: cannot use a (type Address) as type Describer in assignment: Address does not implement Describer (Describe method has pointer receiver)`。这是因为在第 22 行，我们使用 `Address` 类型的指针接受者实现了接口 `Describer`，而接下来我们试图用 `a` 来赋值 `d2`。然而 `a` 属于值类型，它并没有实现 `Describer` 接口。你应该会很惊讶，因为我们曾经学习过，使用指针接受者的方法，无论指针还是值都可以调用它。那么为什么第 45 行的代码就不管用呢？

**其原因是：对于使用指针接受者的方法，用一个指针或者一个可取得地址的值来调用都是合法的。但接口中存储的具体值（Concrete Value）并不能取到地址，因此在第 45 行，对于编译器无法自动获取 a 的地址，于是程序报错**。

第 47 行就可以成功运行，因为我们将 `a` 的地址 `&a` 赋值给了 `d2`。

程序的其他部分不言而喻。该程序会打印：

```
CopySam is 25 years old  
James is 32 years old  
State Washington Country USA
```

### 实现多个接口

类型可以实现多个接口。我们看看下面程序是如何做到的。

```go
Copypackage main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var s SalaryCalculator = e
    s.DisplaySalary()
    var l LeaveCalculator = e
    fmt.Println("\nLeaves left =", l.CalculateLeavesLeft())
}
```

上述程序在第 7 行和第 11 行分别声明了两个接口：`SalaryCalculator` 和 `LeaveCalculator`。

第 15 行定义了结构体 `Employee`，它在第 24 行实现了 `SalaryCalculator` 接口的 `DisplaySalary` 方法，接着在第 28 行又实现了 `LeaveCalculator` 接口里的 `CalculateLeavesLeft` 方法。于是 `Employee` 就实现了 `SalaryCalculator` 和 `LeaveCalculator` 两个接口。

第 41 行，我们把 `e` 赋值给了 `SalaryCalculator` 类型的接口变量 ，而在 43 行，我们同样把 `e` 赋值给 `LeaveCalculator` 类型的接口变量 。由于 `e` 的类型 `Employee` 实现了 `SalaryCalculator` 和 `LeaveCalculator` 两个接口，因此这是合法的。

该程序会输出：

```
CopyNaveen Ramanathan has salary $5200  
Leaves left = 25
```

### 接口的嵌套

尽管 Go 语言没有提供继承机制，但可以通过嵌套其他的接口，创建一个新接口。

我们来看看这如何实现。

```go
Copypackage main

import (  
    "fmt"
)

type SalaryCalculator interface {  
    DisplaySalary()
}

type LeaveCalculator interface {  
    CalculateLeavesLeft() int
}

type EmployeeOperations interface {  
    SalaryCalculator
    LeaveCalculator
}

type Employee struct {  
    firstName string
    lastName string
    basicPay int
    pf int
    totalLeaves int
    leavesTaken int
}

func (e Employee) DisplaySalary() {  
    fmt.Printf("%s %s has salary $%d", e.firstName, e.lastName, (e.basicPay + e.pf))
}

func (e Employee) CalculateLeavesLeft() int {  
    return e.totalLeaves - e.leavesTaken
}

func main() {  
    e := Employee {
        firstName: "Naveen",
        lastName: "Ramanathan",
        basicPay: 5000,
        pf: 200,
        totalLeaves: 30,
        leavesTaken: 5,
    }
    var empOp EmployeeOperations = e
    empOp.DisplaySalary()
    fmt.Println("\nLeaves left =", empOp.CalculateLeavesLeft())
}
```

在上述程序的第 15 行，我们创建了一个新的接口 `EmployeeOperations`，它嵌套了两个接口：`SalaryCalculator` 和 `LeaveCalculator`。

如果一个类型定义了 `SalaryCalculator` 和 `LeaveCalculator` 接口里包含的方法，我们就称该类型实现了 `EmployeeOperations` 接口。

在第 29 行和第 33 行，由于 `Employee` 结构体定义了 `DisplaySalary` 和 `CalculateLeavesLeft` 方法，因此它实现了接口 `EmployeeOperations`。

在 46 行，`empOp` 的类型是 `EmployeeOperations`，`e` 的类型是 `Employee`，我们把 `empOp` 赋值为 `e`。接下来的两行，`empOp` 调用了 `DisplaySalary()` 和 `CalculateLeavesLeft()` 方法。

该程序输出：

```
CopyNaveen Ramanathan has salary $5200
Leaves left = 25
```

### 接口的零值

接口的零值是 `nil`。对于值为 `nil` 的接口，其底层值（Underlying Value）和具体类型（Concrete Type）都为 `nil`。

```go
Copypackage main

import "fmt"

type Describer interface {  
    Describe()
}

func main() {  
    var d1 Describer
    if d1 == nil {
        fmt.Printf("d1 is nil and has type %T value %v\n", d1, d1)
    }
}
```

上面程序里的 `d1` 等于 `nil`，程序会输出：

```
Copyd1 is nil and has type <nil> value <nil>
```

对于值为 `nil` 的接口，由于没有底层值和具体类型，当我们试图调用它的方法时，程序会产生 `panic` 异常。

```go
Copypackage main

type Describer interface {
    Describe()
}

func main() {  
    var d1 Describer
    d1.Describe()
}
```

在上述程序中，`d1` 等于 `nil`，程序产生运行时错误 `panic`： **panic: runtime error: invalid memory address or nil pointer dereference [signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xc8527]** 。
