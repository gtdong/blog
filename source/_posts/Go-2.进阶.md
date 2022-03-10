---
title:  Go-进阶
tags:
  - Go
categories:
  - Go语言
date: 2021-08-20 13:05:00
---


# 20.并发入门

**Go 是并发式语言，而不是并行式语言**。在讨论 Go 如何处理并发之前，我们必须理解何为并发，以及并发与并行的区别。

<!--more-->

## 并发是什么？

并发是指立即处理多个任务的能力。一个例子就能很好地说明这一点。

我们可以想象一个人正在跑步。假如在他晨跑时，鞋带突然松了。于是他停下来，系一下鞋带，接下来继续跑。这个例子就是典型的并发。这个人能够一下搞定跑步和系鞋带两件事，即立即处理多个任务。

## 并行是什么？并行和并发有何区别？

并行是指同时处理多个任务。这听起来和并发差不多，但其实完全不同。

我们同样用这个跑步的例子来帮助理解。假如这个人在慢跑时，还在用他的 iPod 听着音乐。在这里，他是在跑步的同时听音乐，也就是同时处理多个任务。这称之为并行。

## 从技术上看并发和并行

通过现实中的例子，我们已经明白了什么是并发，以及并发与并行的区别。作为一名极客，我们接下来从技术的角度来考察并发和并行。😃

假如我们正在编写一个 web 浏览器。这个 web 浏览器有各种组件。其中两个分别是 web 页面的渲染区和从网上下载文件的下载器。假设我们已经构建好了浏览器代码，各个组件也都可以相互独立地运行（通过像 Java 里的线程，或者通过即将介绍的 Go 语言中的 Go 协程来实现）。当浏览器在单核处理器中运行时，处理器会在浏览器的两个组件间进行上下文切换。它可能在一段时间内下载文件，转而又对用户请求的 web 页面进行渲染。这就是并发。并发的进程从不同的时间点开始，分别交替运行。在这里，就是在不同的时间点开始进行下载和渲染，并相互交替运行的。

如果该浏览器在一个多核处理器上运行，此时下载文件的组件和渲染 HTML 的组件可能会在不同的核上同时运行。这称之为并行。

[![image](https://raw.githubusercontent.com/studygolang/gctt-images/master/golang-series/concurrency-parallelism-copy.png)](https://raw.githubusercontent.com/studygolang/gctt-images/master/golang-series/concurrency-parallelism-copy.png)

并行不一定会加快运行速度，因为并行运行的组件之间可能需要相互通信。在我们浏览器的例子里，当文件下载完成后，应当对用户进行提醒，比如弹出一个窗口。于是，在负责下载的组件和负责渲染用户界面的组件之间，就产生了通信。在并发系统上，这种通信开销很小。但在多核的并行系统上，组件间的通信开销就很高了。所以，并行不一定会加快运行速度！

## Go 对并发的支持

Go 编程语言原生支持并发。Go 使用 Go 协程（Goroutine） 和信道（Channel）来处理并发。在接下来的教程里，我们还会详细介绍它们。

# 21.Go携程

## Go 协程是什么？

Go 协程是与其他函数或方法一起并发运行的函数或方法。Go 协程可以看作是轻量级线程。与线程相比，创建一个 Go 协程的成本很小。因此在 Go 应用中，常常会看到有数以千计的 Go 协程并发地运行。

## Go 协程相比于线程的优势

- 相比线程而言，Go 协程的成本极低。堆栈大小只有若干 kb，并且可以根据应用的需求进行增减。而线程必须指定堆栈的大小，其堆栈是固定不变的。
- Go 协程会复用（Multiplex）数量更少的 OS 线程。即使程序有数以千计的 Go 协程，也可能只有一个线程。如果该线程中的某一 Go 协程发生了阻塞（比如说等待用户输入），那么系统会再创建一个 OS 线程，并把其余 Go 协程都移动到这个新的 OS 线程。所有这一切都在运行时进行，作为程序员，我们没有直接面临这些复杂的细节，而是有一个简洁的 API 来处理并发。
- Go 协程使用信道（Channel）来进行通信。信道用于防止多个协程访问共享内存时发生竞态条件（Race Condition）。信道可以看作是 Go 协程之间通信的管道。我们会在下一教程详细讨论信道。

## 如何启动一个 Go 协程？

调用函数或者方法时，在前面加上关键字 `go`，可以让一个新的 Go 协程并发地运行。

让我们创建一个 Go 协程吧。

```go
Copypackage main

import (
    "fmt"
)

func hello() {
    fmt.Println("Hello world goroutine")
}
func main() {
    go hello()
    fmt.Println("main function")
}
```

在第 11 行，`go hello()` 启动了一个新的 Go 协程。现在 `hello()` 函数与 `main()` 函数会并发地执行。主函数会运行在一个特有的 Go 协程上，它称为 Go 主协程（Main Goroutine）。

**运行一下程序，你会很惊讶！**

该程序只会输出文本 `main function`。我们启动的 Go 协程究竟出现了什么问题？要理解这一切，我们需要理解两个 Go 协程的主要性质。

- **启动一个新的协程时，协程的调用会立即返回。与函数不同，程序控制不会去等待 Go 协程执行完毕。在调用 Go 协程之后，程序控制会立即返回到代码的下一行，忽略该协程的任何返回值。**
- **如果希望运行其他 Go 协程，Go 主协程必须继续运行着。如果 Go 主协程终止，则程序终止，于是其他 Go 协程也不会继续运行。**

现在你应该能够理解，为何我们的 Go 协程没有运行了吧。在第 11 行调用了 `go hello()` 之后，程序控制没有等待 `hello` 协程结束，立即返回到了代码下一行，打印 `main function`。接着由于没有其他可执行的代码，Go 主协程终止，于是 `hello` 协程就没有机会运行了。

我们现在修复这个问题。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```

在上面程序的第 13 行，我们调用了 time 包里的函数 [`Sleep`]，该函数会休眠执行它的 Go 协程。在这里，我们使 Go 主协程休眠了 1 秒。因此在主协程终止之前，调用 `go hello()` 就有足够的时间来执行了。该程序首先打印 `Hello world goroutine`，等待 1 秒钟之后，接着打印 `main function`。

在 Go 主协程中使用休眠，以便等待其他协程执行完毕，这种方法只是用于理解 Go 协程如何工作的技巧。信道可用于在其他协程结束执行之前，阻塞 Go 主协程。我们会在下一教程中讨论信道。

## 启动多个 Go 协程

为了更好地理解 Go 协程，我们再编写一个程序，启动多个 Go 协程。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func numbers() {  
    for i := 1; i <= 5; i++ {
        time.Sleep(250 * time.Millisecond)
        fmt.Printf("%d ", i)
    }
}
func alphabets() {  
    for i := 'a'; i <= 'e'; i++ {
        time.Sleep(400 * time.Millisecond)
        fmt.Printf("%c ", i)
    }
}
func main() {  
    go numbers()
    go alphabets()
    time.Sleep(3000 * time.Millisecond)
    fmt.Println("main terminated")
}
```

在上面程序中的第 21 行和第 22 行，启动了两个 Go 协程。现在，这两个协程并发地运行。`numbers` 协程首先休眠 250 微秒，接着打印 `1`，然后再次休眠，打印 `2`，依此类推，一直到打印 `5` 结束。`alphabete` 协程同样打印从 `a` 到 `e` 的字母，并且每次有 400 微秒的休眠时间。 Go 主协程启动了 `numbers` 和 `alphabete` 两个 Go 协程，休眠了 3000 微秒后终止程序。

该程序会输出：

```
Copy1 a 2 3 b 4 c 5 d e main terminated
```

程序的运作如下图所示

[![img](https://img2018.cnblogs.com/blog/1825659/201910/1825659-20191023232812015-496811664.png)](https://img2018.cnblogs.com/blog/1825659/201910/1825659-20191023232812015-496811664.png)

第一张蓝色的图表示 `numbers` 协程，第二张褐红色的图表示 `alphabets` 协程，第三张绿色的图表示 Go 主协程，而最后一张黑色的图把以上三种协程合并了，表明程序是如何运行的。在每个方框顶部，诸如 `0 ms` 和 `250 ms` 这样的字符串表示时间（以微秒为单位）。在每个方框的底部，`1`、`2`、`3` 等表示输出。蓝色方框表示：`250 ms` 打印出 `1`，`500 ms` 打印出 `2`，依此类推。最后黑色方框的底部的值会是 `1 a 2 3 b 4 c 5 d e main terminated`，这同样也是整个程序的输出。以上图片非常直观，你可以用它来理解程序是如何运作的。

# 22. 信道（channel）

## 什么是信道？

信道可以想像成 Go 协程之间通信的管道。如同管道中的水会从一端流到另一端，通过使用信道，数据也可以从一端发送，在另一端接收。

## 信道的声明

所有信道都关联了一个类型。信道只能运输这种类型的数据，而运输其他类型的数据都是非法的。

`chan T` 表示 `T` 类型的信道。

信道的零值为 `nil`。信道的零值没有什么用，应该像对 map 和切片所做的那样，用 `make` 来定义信道。

下面编写代码，声明一个信道。

```go
Copypackage main

import "fmt"

func main() {  
    var a chan int
    if a == nil {
        fmt.Println("channel a is nil, going to define it")
        a = make(chan int)
        fmt.Printf("Type of a is %T", a)
    }
}
```

由于信道的零值为 `nil`，在第 6 行，信道 `a` 的值就是 `nil`。于是，程序执行了 if 语句内的语句，定义了信道 `a`。程序中 `a` 是一个 int 类型的信道。该程序会输出：

```
Copychannel a is nil, going to define it  
Type of a is chan int
```

简短声明通常也是一种定义信道的简洁有效的方法。

```go
Copya := make(chan int)
```

这一行代码同样定义了一个 int 类型的信道 `a`。

## 通过信道进行发送和接收

如下所示，该语法通过信道发送和接收数据。

```go
Copydata := <- a // 读取信道 a  
a <- data // 写入信道 a
```

信道旁的箭头方向指定了是发送数据还是接收数据。

在第一行，箭头对于 `a` 来说是向外指的，因此我们读取了信道 `a` 的值，并把该值存储到变量 `data`。

在第二行，箭头指向了 `a`，因此我们在把数据写入信道 `a`。

## 发送与接收默认是阻塞的

发送与接收默认是阻塞的。这是什么意思？当把数据发送到信道时，程序控制会在发送数据的语句处发生阻塞，直到有其它 Go 协程从信道读取到数据，才会解除阻塞。与此类似，当读取信道的数据时，如果没有其它的协程把数据写入到这个信道，那么读取过程就会一直阻塞着。

信道的这种特性能够帮助 Go 协程之间进行高效的通信，不需要用到其他编程语言常见的显式锁或条件变量。

## 信道的代码示例

理论已经够了:)。接下来写点代码，看看协程之间通过信道是怎么通信的吧。

我们其实可以重写上章学习 [Go 协程]时写的程序，现在我们在这里用上信道。

首先引用前面教程里的程序。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func hello() {  
    fmt.Println("Hello world goroutine")
}
func main() {  
    go hello()
    time.Sleep(1 * time.Second)
    fmt.Println("main function")
}
```

这是上一篇的代码。我们使用到了休眠，使 Go 主协程等待 hello 协程结束。如果你看不懂，建议你阅读上一教程 [Go 协程]。

我们接下来使用信道来重写上面代码。

```go
Copypackage main

import (  
    "fmt"
)

func hello(done chan bool) {  
    fmt.Println("Hello world goroutine")
    done <- true
}
func main() {  
    done := make(chan bool)
    go hello(done)
    <-done
    fmt.Println("main function")
}
```

在上述程序里，我们在第 12 行创建了一个 bool 类型的信道 `done`，并把 `done` 作为参数传递给了 `hello` 协程。在第 14 行，我们通过信道 `done` 接收数据。这一行代码发生了阻塞，除非有协程向 `done` 写入数据，否则程序不会跳到下一行代码。于是，这就不需要用以前的 `time.Sleep` 来阻止 Go 主协程退出了。

`<-done` 这行代码通过协程（译注：原文笔误，信道）`done` 接收数据，但并没有使用数据或者把数据存储到变量中。这完全是合法的。

现在我们的 Go 主协程发生了阻塞，等待信道 `done` 发送的数据。该信道作为参数传递给了协程 `hello`，`hello` 打印出 `Hello world goroutine`，接下来向 `done` 写入数据。当完成写入时，Go 主协程会通过信道 `done` 接收数据，于是它解除阻塞状态，打印出文本 `main function`。

该程序输出如下：

```
CopyHello world goroutine  
main function
```

我们稍微修改一下程序，在 `hello` 协程里加入休眠函数，以便更好地理解阻塞的概念。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func hello(done chan bool) {  
    fmt.Println("hello go routine is going to sleep")
    time.Sleep(4 * time.Second)
    fmt.Println("hello go routine awake and going to write to done")
    done <- true
}
func main() {  
    done := make(chan bool)
    fmt.Println("Main going to call hello go goroutine")
    go hello(done)
    <-done
    fmt.Println("Main received data")
}
```

在上面程序里，我们向 `hello` 函数里添加了 4 秒的休眠（第 10 行）。

程序首先会打印 `Main going to call hello go goroutine`。接着会开启 `hello` 协程，打印 `hello go routine is going to sleep`。打印完之后，`hello` 协程会休眠 4 秒钟，而在这期间，主协程会在 `<-done` 这一行发生阻塞，等待来自信道 `done` 的数据。4 秒钟之后，打印 `hello go routine awake and going to write to done`，接着再打印 `Main received data`。

## 信道的另一个示例

我们再编写一个程序来更好地理解信道。该程序会计算一个数中每一位的平方和与立方和，然后把平方和与立方和相加并打印出来。

例如，如果输出是 123，该程序会如下计算输出：

```
Copysquares = (1 * 1) + (2 * 2) + (3 * 3) 
cubes = (1 * 1 * 1) + (2 * 2 * 2) + (3 * 3 * 3) 
output = squares + cubes = 50
```

我们会这样去构建程序：在一个单独的 Go 协程计算平方和，而在另一个协程计算立方和，最后在 Go 主协程把平方和与立方和相加。

```go
Copypackage main

import (  
    "fmt"
)

func calcSquares(number int, squareop chan int) {  
    sum := 0
    for number != 0 {
        digit := number % 10
        sum += digit * digit
        number /= 10
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0 
    for number != 0 {
        digit := number % 10
        sum += digit * digit * digit
        number /= 10
    }
    cubeop <- sum
} 

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares + cubes)
}
```

在第 7 行，函数 `calcSquares` 计算一个数每位的平方和，并把结果发送给信道 `squareop`。与此类似，在第 17 行函数 `calcCubes` 计算一个数每位的立方和，并把结果发送给信道 `cubop`。

这两个函数分别在单独的协程里运行（第 31 行和第 32 行），每个函数都有传递信道的参数，以便写入数据。Go 主协程会在第 33 行等待两个信道传来的数据。一旦从两个信道接收完数据，数据就会存储在变量 `squares` 和 `cubes` 里，然后计算并打印出最后结果。该程序会输出：

```
CopyFinal output 1536
```

## 死锁

使用信道需要考虑的一个重点是死锁。当 Go 协程给一个信道发送数据时，照理说会有其他 Go 协程来接收数据。如果没有的话，程序就会在运行时触发 panic，形成死锁。

同理，当有 Go 协程等着从一个信道接收数据时，我们期望其他的 Go 协程会向该信道写入数据，要不然程序就会触发 panic。

```go
Copypackage main

func main() {  
    ch := make(chan int)
    ch <- 5
}
```

在上述程序中，我们创建了一个信道 `ch`，接着在下一行 `ch <- 5`，我们把 `5` 发送到这个信道。对于本程序，没有其他的协程从 `ch` 接收数据。于是程序触发 panic，出现如下运行时错误。

```
Copyfatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox249677995/main.go:6 +0x80
```

## 单向信道

我们目前讨论的信道都是双向信道，即通过信道既能发送数据，又能接收数据。其实也可以创建单向信道，这种信道只能发送或者接收数据。

```go
Copypackage main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    sendch := make(chan<- int)
    go sendData(sendch)
    fmt.Println(<-sendch)
}
```

上面程序的第 10 行，我们创建了唯送（Send Only）信道 `sendch`。`chan<- int` 定义了唯送信道，因为箭头指向了 `chan`。在第 12 行，我们试图通过唯送信道接收数据，于是编译器报错：

```
Copymain.go:11: invalid operation: <-sendch (receive from send-only type chan<- int)
```

**一切都很顺利，只不过一个不能读取数据的唯送信道究竟有什么意义呢？**

**这就需要用到信道转换（Channel Conversion）了。把一个双向信道转换成唯送信道或者唯收（Receive Only）信道都是行得通的，但是反过来就不行。**

```go
Copypackage main

import "fmt"

func sendData(sendch chan<- int) {  
    sendch <- 10
}

func main() {  
    cha1 := make(chan int)
    go sendData(cha1)
    fmt.Println(<-cha1)
}
```

在上述程序的第 10 行，我们创建了一个双向信道 `cha1`。在第 11 行 `cha1` 作为参数传递给了 `sendData` 协程。在第 5 行，函数 `sendData` 里的参数 `sendch chan<- int` 把 `cha1` 转换为一个唯送信道。于是该信道在 `sendData` 协程里是一个唯送信道，而在 Go 主协程里是一个双向信道。该程序最终打印输出 `10`。

## 关闭信道和使用 for range 遍历信道

数据发送方可以关闭信道，通知接收方这个信道不再有数据发送过来。

当从信道接收数据时，接收方可以多用一个变量来检查信道是否已经关闭。

```
Copyv, ok := <- ch
```

上面的语句里，如果成功接收信道所发送的数据，那么 `ok` 等于 true。而如果 `ok` 等于 false，说明我们试图读取一个关闭的通道。从关闭的信道读取到的值会是该信道类型的零值。例如，当信道是一个 `int` 类型的信道时，那么从关闭的信道读取的值将会是 `0`。

```go
Copypackage main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {  
    ch := make(chan int)
    go producer(ch)
    for {
        v, ok := <-ch
        if ok == false {
            break
        }
        fmt.Println("Received ", v, ok)
    }
}
```

在上述的程序中，`producer` 协程会从 0 到 9 写入信道 `chn1`，然后关闭该信道。主函数有一个无限的 for 循环（第 16 行），使用变量 `ok`（第 18 行）检查信道是否已经关闭。如果 `ok` 等于 false，说明信道已经关闭，于是退出 for 循环。如果 `ok` 等于 true，会打印出接收到的值和 `ok` 的值。

```
CopyReceived  0 true  
Received  1 true  
Received  2 true  
Received  3 true  
Received  4 true  
Received  5 true  
Received  6 true  
Received  7 true  
Received  8 true  
Received  9 true
```

for range 循环用于在一个信道关闭之前，从信道接收数据。

接下来我们使用 for range 循环重写上面的代码。

```go
Copypackage main

import (  
    "fmt"
)

func producer(chnl chan int) {  
    for i := 0; i < 10; i++ {
        chnl <- i
    }
    close(chnl)
}
func main() {  
    ch := make(chan int)
    go producer(ch)
    for v := range ch {
        fmt.Println("Received ",v)
    }
}
```

在第 16 行，for range 循环从信道 `ch` 接收数据，直到该信道关闭。一旦关闭了 `ch`，循环会自动结束。该程序会输出：

```
CopyReceived  0  
Received  1  
Received  2  
Received  3  
Received  4  
Received  5  
Received  6  
Received  7  
Received  8  
Received  9
```

我们可以使用 for range 循环，重写[信道的另一个示例]这一节里面的代码，提高代码的可重用性。

如果你仔细观察这段代码，会发现获得一个数里的每位数的代码在 `calcSquares` 和 `calcCubes` 两个函数内重复了。我们将把这段代码抽离出来，放在一个单独的函数里，然后并发地调用它。

```go
Copypackage main

import (  
    "fmt"
)

func digits(number int, dchnl chan int) {  
    for number != 0 {
        digit := number % 10
        dchnl <- digit
        number /= 10
    }
    close(dchnl)
}
func calcSquares(number int, squareop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit
    }
    squareop <- sum
}

func calcCubes(number int, cubeop chan int) {  
    sum := 0
    dch := make(chan int)
    go digits(number, dch)
    for digit := range dch {
        sum += digit * digit * digit
    }
    cubeop <- sum
}

func main() {  
    number := 589
    sqrch := make(chan int)
    cubech := make(chan int)
    go calcSquares(number, sqrch)
    go calcCubes(number, cubech)
    squares, cubes := <-sqrch, <-cubech
    fmt.Println("Final output", squares+cubes)
}
```

上述程序里的 `digits` 函数，包含了获取一个数的每位数的逻辑，并且 `calcSquares` 和 `calcCubes` 两个函数并发地调用了 `digits`。当计算完数字里面的每一位数时，第 13 行就会关闭信道。`calcSquares` 和 `calcCubes` 两个协程使用 for range 循环分别监听了它们的信道，直到该信道关闭。程序的其他地方不变，该程序同样会输出：

```
CopyFinal output 1536
```

关于信道还有一些其他的概念，比如缓冲信道（Buffered Channel）、工作池（Worker Pool）和 select。我们会在接下来的教程里专门介绍它们

# 23. 缓冲信道和工作池（Buffered Channels and Worker Pools）

## 什么是缓冲信道？

在[上一教程]里，我们讨论的主要是无缓冲信道。我们在[信道]的教程里详细讨论了，无缓冲信道的发送和接收过程是阻塞的。

我们还可以创建一个有缓冲（Buffer）的信道。只在缓冲已满的情况，才会阻塞向缓冲信道（Buffered Channel）发送数据。同样，只有在缓冲为空的时候，才会阻塞从缓冲信道接收数据。

通过向 `make` 函数再传递一个表示容量的参数（指定缓冲的大小），可以创建缓冲信道。

```go
Copych := make(chan type, capacity)
```

要让一个信道有缓冲，上面语法中的 `capacity` 应该大于 0。无缓冲信道的容量默认为 0，因此我们在[上一教程]创建信道时，省略了容量参数。

我们开始编写代码，创建一个缓冲信道。

## 示例一

```go
Copypackage main

import (  
    "fmt"
)


func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println(<- ch)
    fmt.Println(<- ch)
}
```

在上面程序里的第 9 行，我们创建了一个缓冲信道，其容量为 2。由于该信道的容量为 2，因此可向它写入两个字符串，而且不会发生阻塞。在第 10 行和第 11 行，我们向信道写入两个字符串，该信道并没有发生阻塞。我们又在第 12 行和第 13 行分别读取了这两个字符串。该程序输出：

```
Copynaveen  
paul
```

## 示例二

我们再看一个缓冲信道的示例，其中有一个并发的 Go 协程来向信道写入数据，而 Go 主协程负责读取数据。该示例帮助我们进一步理解，在向缓冲信道写入数据时，什么时候会发生阻塞。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func write(ch chan int) {  
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Println("successfully wrote", i, "to ch")
    }
    close(ch)
}
func main() {  
    ch := make(chan int, 2)
    go write(ch)
    time.Sleep(2 * time.Second)
    for v := range ch {
        fmt.Println("read value", v,"from ch")
        time.Sleep(2 * time.Second)

    }
}
```

在上面的程序中，第 16 行在 Go 主协程中创建了容量为 2 的缓冲信道 `ch`，而第 17 行把 `ch` 传递给了 `write` 协程。接下来 Go 主协程休眠了两秒。在这期间，`write` 协程在并发地运行。`write` 协程有一个 for 循环，依次向信道 `ch` 写入 0～4。而缓冲信道的容量为 2，因此 `write` 协程里立即会向 `ch` 写入 0 和 1，接下来发生阻塞，直到 `ch` 内的值被读取。因此，该程序立即打印出下面两行：

```
Copysuccessfully wrote 0 to ch  
successfully wrote 1 to ch
```

打印上面两行之后，`write` 协程中向 `ch` 的写入发生了阻塞，直到 `ch` 有值被读取到。而 Go 主协程休眠了两秒后，才开始读取该信道，因此在休眠期间程序不会打印任何结果。主协程结束休眠后，在第 19 行使用 for range 循环，开始读取信道 `ch`，打印出了读取到的值后又休眠两秒，这个循环一直到 `ch` 关闭才结束。所以该程序在两秒后会打印下面两行：

```
Copyread value 0 from ch  
successfully wrote 2 to ch
```

该过程会一直进行，直到信道读取完所有的值，并在 `write` 协程中关闭信道。最终输出如下：

```
Copysuccessfully wrote 0 to ch  
successfully wrote 1 to ch  
read value 0 from ch  
successfully wrote 2 to ch  
read value 1 from ch  
successfully wrote 3 to ch  
read value 2 from ch  
successfully wrote 4 to ch  
read value 3 from ch  
read value 4 from ch
```

## 死锁

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 2)
    ch <- "naveen"
    ch <- "paul"
    ch <- "steve"
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

在上面程序里，我们向容量为 2 的缓冲信道写入 3 个字符串。当在程序控制到达第 3 次写入时（第 11 行），由于它超出了信道的容量，因此这次写入发生了阻塞。现在想要这次写操作能够进行下去，必须要有其它协程来读取这个信道的数据。但在本例中，并没有并发协程来读取这个信道，因此这里会发生**死锁**（deadlock）。程序会在运行时触发 panic，信息如下：

```
Copyfatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:  
main.main()  
    /tmp/sandbox274756028/main.go:11 +0x100
```

## 长度 vs 容量

缓冲信道的容量是指信道可以存储的值的数量。我们在使用 `make` 函数创建缓冲信道的时候会指定容量大小。

缓冲信道的长度是指信道中当前排队的元素个数。

代码可以把一切解释得很清楚。😃

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    ch := make(chan string, 3)
    ch <- "naveen"
    ch <- "paul"
    fmt.Println("capacity is", cap(ch))
    fmt.Println("length is", len(ch))
    fmt.Println("read value", <-ch)
    fmt.Println("new length is", len(ch))
}
```

在上面的程序里，我们创建了一个容量为 3 的信道，于是它可以保存 3 个字符串。接下来，我们分别在第 9 行和第 10 行向信道写入了两个字符串。于是信道有两个字符串排队，因此其长度为 2。在第 13 行，我们又从信道读取了一个字符串。现在该信道内只有一个字符串，因此其长度变为 1。该程序会输出：

```
Copycapacity is 3  
length is 2  
read value naveen  
new length is 1
```

## WaitGroup

在本教程的下一节里，我们会讲到**工作池**（Worker Pools）。而 `WaitGroup` 用于实现工作池，因此要理解工作池，我们首先需要学习 `WaitGroup`。

`WaitGroup` 用于等待一批 Go 协程执行结束。程序控制会一直阻塞，直到这些协程全部执行完毕。假设我们有 3 个并发执行的 Go 协程（由 Go 主协程生成）。Go 主协程需要等待这 3 个协程执行结束后，才会终止。这就可以用 `WaitGroup` 来实现。

理论说完了，我们编写点儿代码吧。😃

```go
Copypackage main

import (  
    "fmt"
    "sync"
    "time"
)

func process(i int, wg *sync.WaitGroup) {  
    fmt.Println("started Goroutine ", i)
    time.Sleep(2 * time.Second)
    fmt.Printf("Goroutine %d ended\n", i)
    wg.Done()
}

func main() {  
    no := 3
    var wg sync.WaitGroup
    for i := 0; i < no; i++ {
        wg.Add(1)
        go process(i, &wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

[WaitGroup]是一个结构体类型，我们在第 18 行创建了 `WaitGroup` 类型的变量，其初始值为零值。`WaitGroup` 使用计数器来工作。当我们调用 `WaitGroup` 的 `Add` 并传递一个 `int` 时，`WaitGroup` 的计数器会加上 `Add` 的传参。要减少计数器，可以调用 `WaitGroup` 的 `Done()` 方法。`Wait()` 方法会阻塞调用它的 Go 协程，直到计数器变为 0 后才会停止阻塞。

上述程序里，for 循环迭代了 3 次，我们在循环内调用了 `wg.Add(1)`（第 20 行）。因此计数器变为 3。for 循环同样创建了 3 个 `process` 协程，然后在第 23 行调用了 `wg.Wait()`，确保 Go 主协程等待计数器变为 0。在第 13 行，`process` 协程内调用了 `wg.Done`，可以让计数器递减。一旦 3 个子协程都执行完毕（即 `wg.Done()` 调用了 3 次），那么计数器就变为 0，于是主协程会解除阻塞。

**在第 21 行里，传递 wg 的地址是很重要的。如果没有传递 wg 的地址，那么每个 Go 协程将会得到一个 WaitGroup 值的拷贝，因而当它们执行结束时，main 函数并不会知道**。

该程序输出：

```
Copystarted Goroutine  2  
started Goroutine  0  
started Goroutine  1  
Goroutine 0 ended  
Goroutine 2 ended  
Goroutine 1 ended  
All go routines finished executing
```

由于 Go 协程的执行顺序不一定，因此你的输出可能和我不一样。😃

## 工作池的实现

缓冲信道的重要应用之一就是实现[工作池]。

一般而言，工作池就是一组等待任务分配的线程。一旦完成了所分配的任务，这些线程可继续等待任务的分配。

我们会使用缓冲信道来实现工作池。我们工作池的任务是计算所输入数字的每一位的和。例如，如果输入 234，结果会是 9（即 2 + 3 + 4）。向工作池输入的是一列伪随机数。

我们工作池的核心功能如下：

- 创建一个 Go 协程池，监听一个等待作业分配的输入型缓冲信道。
- 将作业添加到该输入型缓冲信道中。
- 作业完成后，再将结果写入一个输出型缓冲信道。
- 从输出型缓冲信道读取并打印结果。

我们会逐步编写这个程序，让代码易于理解。

第一步就是创建一个结构体，表示作业和结果。

```go
Copytype Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}
```

所有 `Job` 结构体变量都会有 `id` 和 `randomno` 两个字段，`randomno` 用于计算其每位数之和。

而 `Result` 结构体有一个 `job` 字段，表示所对应的作业，还有一个 `sumofdigits` 字段，表示计算的结果（每位数字之和）。

第二步是分别创建用于接收作业和写入结果的缓冲信道。

```go
Copyvar jobs = make(chan Job, 10)  
var results = make(chan Result, 10)
```

工作协程（Worker Goroutine）会监听缓冲信道 `jobs` 里更新的作业。一旦工作协程完成了作业，其结果会写入缓冲信道 `results`。

如下所示，`digits` 函数的任务实际上就是计算整数的每一位之和，最后返回该结果。为了模拟出 `digits` 在计算过程中花费了一段时间，我们在函数内添加了两秒的休眠时间。

```go
Copyfunc digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
```

然后，我们写一个创建工作协程的函数。

```go
Copyfunc worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
```

上面的函数创建了一个工作者（Worker），读取 `jobs` 信道的数据，根据当前的 `job` 和 `digits` 函数的返回值，创建了一个 `Result` 结构体变量，然后将结果写入 `results` 缓冲信道。`worker` 函数接收了一个 `WaitGroup` 类型的 `wg` 作为参数，当所有的 `jobs` 完成的时候，调用了 `Done()` 方法。

`createWorkerPool` 函数创建了一个 Go 协程的工作池。

```go
Copyfunc createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
```

上面函数的参数是需要创建的工作协程的数量。在创建 Go 协程之前，它调用了 `wg.Add(1)` 方法，于是 `WaitGroup` 计数器递增。接下来，我们创建工作协程，并向 `worker` 函数传递 `wg` 的地址。创建了需要的工作协程后，函数调用 `wg.Wait()`，等待所有的 Go 协程执行完毕。所有协程完成执行之后，函数会关闭 `results` 信道。因为所有协程都已经执行完毕，于是不再需要向 `results` 信道写入数据了。

现在我们已经有了工作池，我们继续编写一个函数，把作业分配给工作者。

```go
Copyfunc allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
```

上面的 `allocate` 函数接收所需创建的作业数量作为输入参数，生成了最大值为 998 的伪随机数，并使用该随机数创建了 `Job` 结构体变量。这个函数把 for 循环的计数器 `i` 作为 id，最后把创建的结构体变量写入 `jobs` 信道。当写入所有的 `job` 时，它关闭了 `jobs` 信道。

下一步是创建一个读取 `results` 信道和打印输出的函数。

```go
Copyfunc result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
```

`result` 函数读取 `results` 信道，并打印出 `job` 的 `id`、输入的随机数、该随机数的每位数之和。`result` 函数也接受 `done` 信道作为参数，当打印所有结果时，`done` 会被写入 true。

现在一切准备充分了。我们继续完成最后一步，在 `main()` 函数中调用上面所有的函数。

```go
Copyfunc main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

我们首先在 `main` 函数的第 2 行，保存了程序的起始时间，并在最后一行（第 12 行）计算了 `endTime` 和 `startTime` 的差值，显示出程序运行的总时间。由于我们想要通过改变协程数量，来做一点基准指标（Benchmark），所以需要这么做。

我们把 `noOfJobs` 设置为 100，接下来调用了 `allocate`，向 `jobs` 信道添加作业。

我们创建了 `done` 信道，并将其传递给 `result` 协程。于是该协程会开始打印结果，并在完成打印时发出通知。

通过调用 `createWorkerPool` 函数，我们最终创建了一个有 10 个协程的工作池。`main` 函数会监听 `done` 信道的通知，等待所有结果打印结束。

为了便于参考，下面是整个程序。我还引用了必要的包。

```go
Copypackage main

import (  
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Job struct {  
    id       int
    randomno int
}
type Result struct {  
    job         Job
    sumofdigits int
}

var jobs = make(chan Job, 10)  
var results = make(chan Result, 10)

func digits(number int) int {  
    sum := 0
    no := number
    for no != 0 {
        digit := no % 10
        sum += digit
        no /= 10
    }
    time.Sleep(2 * time.Second)
    return sum
}
func worker(wg *sync.WaitGroup) {  
    for job := range jobs {
        output := Result{job, digits(job.randomno)}
        results <- output
    }
    wg.Done()
}
func createWorkerPool(noOfWorkers int) {  
    var wg sync.WaitGroup
    for i := 0; i < noOfWorkers; i++ {
        wg.Add(1)
        go worker(&wg)
    }
    wg.Wait()
    close(results)
}
func allocate(noOfJobs int) {  
    for i := 0; i < noOfJobs; i++ {
        randomno := rand.Intn(999)
        job := Job{i, randomno}
        jobs <- job
    }
    close(jobs)
}
func result(done chan bool) {  
    for result := range results {
        fmt.Printf("Job id %d, input random no %d , sum of digits %d\n", result.job.id, result.job.randomno, result.sumofdigits)
    }
    done <- true
}
func main() {  
    startTime := time.Now()
    noOfJobs := 100
    go allocate(noOfJobs)
    done := make(chan bool)
    go result(done)
    noOfWorkers := 10
    createWorkerPool(noOfWorkers)
    <-done
    endTime := time.Now()
    diff := endTime.Sub(startTime)
    fmt.Println("total time taken ", diff.Seconds(), "seconds")
}
```

为了更精确地计算总时间，请在你的本地机器上运行该程序。

该程序输出：

```
CopyJob id 1, input random no 636, sum of digits 15  
Job id 0, input random no 878, sum of digits 23  
Job id 9, input random no 150, sum of digits 6  
...
total time taken  20.01081009 seconds
```

程序总共会打印 100 行，对应着 100 项作业，然后最后会打印一行程序消耗的总时间。你的输出会和我的不同，因为 Go 协程的运行顺序不一定，同样总时间也会因为硬件而不同。在我的例子中，运行程序大约花费了 20 秒。

现在我们把 `main` 函数里的 `noOfWorkers` 增加到 20。我们把工作者的数量加倍了。由于工作协程增加了（准确说来是两倍），因此程序花费的总时间会减少（准确说来是一半）。在我的例子里，程序会打印出 10.004364685 秒。

```
Copy...
total time taken  10.004364685 seconds
```

现在我们可以理解了，随着工作协程数量增加，完成作业的总时间会减少。你们可以练习一下：在 `main` 函数里修改 `noOfJobs` 和 `noOfWorkers` 的值，并试着去分析一下结果。

# 24. Select

## 什么是 select？

`select` 语句用于在多个发送/接收信道操作中进行选择。`select` 语句会一直阻塞，直到发送/接收操作准备就绪。如果有多个信道操作准备完毕，`select` 会随机地选取其中之一执行。该语法与 `switch` 类似，所不同的是，这里的每个 `case` 语句都是信道操作。我们好好看一些代码来加深理解吧。

## 示例

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func server1(ch chan string) {  
    time.Sleep(6 * time.Second)
    ch <- "from server1"
}
func server2(ch chan string) {  
    time.Sleep(3 * time.Second)
    ch <- "from server2"

}
func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

在上面程序里，`server1` 函数（第 8 行）休眠了 6 秒，接着将文本 `from server1` 写入信道 `ch`。而 `server2` 函数（第 12 行）休眠了 3 秒，然后把 `from server2` 写入了信道 `ch`。

而 `main` 函数在第 20 行和第 21 行，分别调用了 `server1` 和 `server2` 两个 Go 协程。

在第 22 行，程序运行到了 `select` 语句。`select` 会一直发生阻塞，除非其中有 case 准备就绪。在上述程序里，`server1` 协程会在 6 秒之后写入 `output1` 信道，而`server2` 协程在 3 秒之后就写入了 `output2` 信道。因此 `select` 语句会阻塞 3 秒钟，等着 `server2` 向 `output2` 信道写入数据。3 秒钟过后，程序会输出：

```
Copyfrom server2
```

然后程序终止。

## select 的应用

在上面程序中，函数之所以取名为 `server1` 和 `server2`，是为了展示 `select` 的实际应用。

假设我们有一个关键性应用，需要尽快地把输出返回给用户。这个应用的数据库复制并且存储在世界各地的服务器上。假设函数 `server1` 和 `server2` 与这样不同区域的两台服务器进行通信。每台服务器的负载和网络时延决定了它的响应时间。我们向两台服务器发送请求，并使用 `select` 语句等待相应的信道发出响应。`select` 会选择首先响应的服务器，而忽略其它的响应。使用这种方法，我们可以向多个服务器发送请求，并给用户返回最快的响应了。:）

## 默认情况

在没有 case 准备就绪时，可以执行 `select` 语句中的默认情况（Default Case）。这通常用于防止 `select` 语句一直阻塞。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func process(ch chan string) {  
    time.Sleep(10500 * time.Millisecond)
    ch <- "process successful"
}

func main() {  
    ch := make(chan string)
    go process(ch)
    for {
        time.Sleep(1000 * time.Millisecond)
        select {
        case v := <-ch:
            fmt.Println("received value: ", v)
            return
        default:
            fmt.Println("no value received")
        }
    }

}
```

上述程序中，第 8 行的 `process` 函数休眠了 10500 毫秒（10.5 秒），接着把 `process successful` 写入 `ch` 信道。在程序中的第 15 行，并发地调用了这个函数。

在并发地调用了 `process` 协程之后，主协程启动了一个无限循环。这个无限循环在每一次迭代开始时，都会先休眠 1000 毫秒（1 秒），然后执行一个 select 操作。在最开始的 10500 毫秒中，由于 `process` 协程在 10500 毫秒后才会向 `ch` 信道写入数据，因此 `select` 语句的第一个 case（即 `case v := <-ch:`）并未就绪。所以在这期间，程序会执行默认情况，该程序会打印 10 次 `no value received`。

在 10.5 秒之后，`process` 协程会在第 10 行向 `ch` 写入 `process successful`。现在，就可以执行 `select` 语句的第一个 case 了，程序会打印 `received value: process successful`，然后程序终止。该程序会输出：

```
Copyno value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
no value received  
received value:  process successful
```

## 死锁与默认情况

```go
Copypackage main

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    }
}
```

上面的程序中，我们在第 4 行创建了一个信道 `ch`。我们在 `select` 内部（第 6 行），试图读取信道 `ch`。由于没有 Go 协程向该信道写入数据，因此 `select` 语句会一直阻塞，导致死锁。该程序会触发运行时 `panic`，报错信息如下：

```
Copyfatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:  
main.main()  
    /tmp/sandbox416567824/main.go:6 +0x80
```

如果存在默认情况，就不会发生死锁，因为在没有其他 case 准备就绪时，会执行默认情况。我们用默认情况重写后，程序如下：

```go
Copypackage main

import "fmt"

func main() {  
    ch := make(chan string)
    select {
    case <-ch:
    default:
        fmt.Println("default case executed")
    }
}
```

以上程序会输出：

```
Copydefault case executed
```

如果 `select` 只含有值为 `nil` 的信道，也同样会执行默认情况。

```go
Copypackage main

import "fmt"

func main() {  
    var ch chan string
    select {
    case v := <-ch:
        fmt.Println("received value", v)
    default:
        fmt.Println("default case executed")

    }
}
```

在上面程序中，`ch` 等于 `nil`，而我们试图在 `select` 中读取 `ch`（第 8 行）。如果没有默认情况，`select` 会一直阻塞，导致死锁。由于我们在 `select` 内部加入了默认情况，程序会执行它，并输出：

```
Copydefault case executed
```

## 随机选取

当 `select` 由多个 case 准备就绪时，将会随机地选取其中之一去执行。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func server1(ch chan string) {  
    ch <- "from server1"
}
func server2(ch chan string) {  
    ch <- "from server2"

}
func main() {  
    output1 := make(chan string)
    output2 := make(chan string)
    go server1(output1)
    go server2(output2)
    time.Sleep(1 * time.Second)
    select {
    case s1 := <-output1:
        fmt.Println(s1)
    case s2 := <-output2:
        fmt.Println(s2)
    }
}
```

在上面程序里，我们在第 18 行和第 19 行分别调用了 `server1` 和 `server2` 两个 Go 协程。接下来，主程序休眠了 1 秒钟（第 20 行）。当程序控制到达第 21 行的 `select` 语句时，`server1` 已经把 `from server1` 写到了 `output1` 信道上，而 `server2` 也同样把 `from server2` 写到了 `output2` 信道上。因此这个 `select` 语句中的两种情况都准备好执行了。如果你运行这个程序很多次的话，输出会是 `from server1` 或者 `from server2`，这会根据随机选取的结果而变化。

请在你的本地系统上运行这个程序，获得程序的随机结果。因为如果你在 playground 上在线运行的话，它的输出总是一样的，这是由于 playground 不具有随机性所造成的。

## 这下我懂了：空 select

```go
Copypackage main

func main() {  
    select {}
}
```

你认为上面代码会输出什么？

我们已经知道，除非有 case 执行，select 语句就会一直阻塞着。在这里，`select` 语句没有任何 case，因此它会一直阻塞，导致死锁。该程序会触发 panic，输出如下：

```
Copyfatal error: all goroutines are asleep - deadlock!

goroutine 1 [select (no cases)]:  
main.main()  
    /tmp/sandbox299546399/main.go:4 +0x20
```

# 26. 结构体取代类

## Go 支持面向对象吗？

Go 并不是完全面向对象的编程语言。Go 官网回答了 Go 是否是面向对象语言，摘录如下。

> 可以说是，也可以说不是。虽然 Go 有类型和方法，支持面向对象的编程风格，但却没有类型的层次结构。Go 中的“接口”概念提供了一种不同的方法，我们认为它易于使用，也更为普遍。Go 也可以将结构体嵌套使用，这与子类化（Subclassing）类似，但并不完全相同。此外，Go 提供的特性比 C++ 或 Java 更为通用：子类可以由任何类型的数据来定义，甚至是内建类型（如简单的“未装箱的”整型）。这在结构体（类）中没有受到限制。

在接下来的教程里，我们会讨论如何使用 Go 来实现面向对象编程概念。与其它面向对象语言（如 Java）相比，Go 有很多完全不同的特性。

## 使用结构体，而非类

Go 不支持类，而是提供了[结构体]。结构体中可以添加[方法]。这样可以将数据和操作数据的方法绑定在一起，实现与类相似的效果。

为了加深理解，我们来编写一个示例吧。

在示例中，我们创建一个自定义[包]，它帮助我们更好地理解，结构体是如何有效地取代类的。

在你的 Go 工作区创建一个名为 `oop` 的文件夹。在 `opp` 中再创建子文件夹 `employee`。在 `employee` 内，创建一个名为 `employee.go` 的文件。

文件夹结构会是这样：

```
Copyworkspacepath -> oop -> employee -> employee.go
```

请将 `employee.go` 里的内容替换为如下所示的代码。

```go
Copypackage employee

import (  
    "fmt"
)

type Employee struct {  
    FirstName   string
    LastName    string
    TotalLeaves int
    LeavesTaken int
}

func (e Employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.FirstName, e.LastName, (e.TotalLeaves - e.LeavesTaken))
}
```

在上述程序里，第 1 行指定了该文件属于 `employee` 包。而第 7 行声明了一个 `Employee` 结构体。在第 14 行，结构体 `Employee` 添加了一个名为 `LeavesRemaining` 的方法。该方法会计算和显示员工的剩余休假数。于是现在我们有了一个结构体，并绑定了结构体的方法，这与类很相似。

接着在 `oop` 文件夹里创建一个文件，命名为 `main.go`。

现在目录结构如下所示：

```
Copyworkspacepath -> oop -> employee -> employee.go  
workspacepath -> oop -> main.go
```

`main.go` 的内容如下所示：

```go
Copypackage main

import "oop/employee"

func main() {  
    e := employee.Employee {
        FirstName: "Sam",
        LastName: "Adolf",
        TotalLeaves: 30,
        LeavesTaken: 20,
    }
    e.LeavesRemaining()
}
```

我们在第 3 行引用了 `employee` 包。在 `main()`（第 12 行），我们调用了 `Employee` 的 `LeavesRemaining()` 方法。

由于有自定义包，这个程序不能在 go playground 上运行。你可以在你的本地运行，在 `workspacepath/bin/oop` 下输入命令 `go install opp`，程序会打印输出：

```bash
CopySam Adolf has 10 leaves remaining
```

## 使用 New() 函数，而非构造器

我们上面写的程序看起来没什么问题，但还是有一些细节问题需要注意。我们看看当定义一个零值的 `employee` 结构体变量时，会发生什么。将 `main.go` 的内容修改为如下代码：

```go
Copypackage main

import "oop/employee"

func main() {  
    var e employee.Employee
    e.LeavesRemaining()
}
```

我们的修改只是创建一个零值的 `Employee` 结构体变量（第 6 行）。该程序会输出：

```bash
Copyhas 0 leaves remaining
```

你可以看到，使用 `Employee` 创建的零值变量没有什么用。它没有合法的姓名，也没有合理的休假细节。

在像 Java 这样的 OOP 语言中，是使用构造器来解决这种问题的。一个合法的对象必须使用参数化的构造器来创建。

Go 并不支持构造器。如果某类型的零值不可用，需要程序员来隐藏该类型，避免从其他包直接访问。程序员应该提供一种名为 `NewT(parameters)` 的 [函数]，按照要求来初始化 `T` 类型的变量。按照 Go 的惯例，应该把创建 `T` 类型变量的函数命名为 `NewT(parameters)`。这就类似于构造器了。如果一个包只含有一种类型，按照 Go 的惯例，应该把函数命名为 `New(parameters)`， 而不是 `NewT(parameters)`。

让我修改一下原先的代码，使得每当创建 `employee` 的时候，它都是可用的。

首先应该让 `Employee` 结构体不可引用，然后创建一个 `New` 函数，用于创建 `Employee` 结构体变量。在 `employee.go` 中输入下面代码：

```go
Copypackage employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

我们进行了一些重要的修改。我们把 `Employee` 结构体的首字母改为小写 `e`，也就是将 `type Employee struct` 改为了 `type employee struct`。通过这种方法，我们把 `employee` 结构体变为了不可引用的，防止其他包对它的访问。除非有特殊需求，否则也要隐藏所有不可引用的结构体的所有字段，这是 Go 的最佳实践。由于我们不会在外部包需要 `employee` 的字段，因此我们也让这些字段无法引用。

同样，我们还修改了 `LeavesRemaining()` 的方法。

现在由于 `employee` 不可引用，因此不能在其他包内直接创建 `Employee` 类型的变量。于是我们在第 14 行提供了一个可引用的 `New` 函数，该函数接收必要的参数，返回一个新创建的 `employee` 结构体变量。

这个程序还需要一些必要的修改，但现在先运行这个程序，理解一下当前的修改。如果运行当前程序，编译器会报错，如下所示：

```bash
Copygo/src/constructor/main.go:6: undefined: employee.Employee
```

这是因为我们将 `Employee` 设置为不可引用，因此编译器会报错，提示该类型没有在 `main.go` 中定义。很完美，正如我们期望的一样，其他包现在不能轻易创建零值的 `employee` 变量了。我们成功地避免了创建不可用的 `employee` 结构体变量。现在创建 `employee` 变量的唯一方法就是使用 `New` 函数。

如下所示，修改 `main.go` 里的内容。

```go
Copypackage main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

该文件唯一的修改就是第 6 行。通过向 `New` 函数传入所需变量，我们创建了一个新的 `employee` 结构体变量。

下面是修改后的两个文件的内容。

employee.go

```go
Copypackage employee

import (  
    "fmt"
)

type employee struct {  
    firstName   string
    lastName    string
    totalLeaves int
    leavesTaken int
}

func New(firstName string, lastName string, totalLeave int, leavesTaken int) employee {  
    e := employee {firstName, lastName, totalLeave, leavesTaken}
    return e
}

func (e employee) LeavesRemaining() {  
    fmt.Printf("%s %s has %d leaves remaining", e.firstName, e.lastName, (e.totalLeaves - e.leavesTaken))
}
```

main.go

```go
Copypackage main  

import "oop/employee"

func main() {  
    e := employee.New("Sam", "Adolf", 30, 20)
    e.LeavesRemaining()
}
```

运行该程序，会输出：

```bash
CopySam Adolf has 10 leaves remaining
```

现在你能明白了，虽然 Go 不支持类，但结构体能够很好地取代类，而以 `New(parameters)` 签名的方法可以替代构造器。

# 27. 组合取代继承

Go 不支持继承，但它支持组合（Composition）。组合一般定义为“合并在一起”。汽车就是一个关于组合的例子：一辆汽车由车轮、引擎和其他各种部件组合在一起。

## 通过嵌套结构体进行组合

在 Go 中，通过在结构体内嵌套结构体，可以实现组合。

组合的典型例子就是博客帖子。每一个博客的帖子都有标题、内容和作者信息。使用组合可以很好地表示它们。通过学习本教程后面的内容，我们会知道如何实现组合。

我们首先创建一个 `author` 结构体。

```go
Copypackage main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}
```

在上面的代码片段中，我们创建了一个 `author` 结构体，`author` 的字段有 `firstname`、`lastname` 和 `bio`。我们还添加了一个 `fullName()` 方法，其中 `author` 作为接收者类型，该方法返回了作者的全名。

下一步我们创建 `post` 结构体。

```go
Copytype post struct {  
    title     string
    content   string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.author.fullName())
    fmt.Println("Bio: ", p.author.bio)
}
```

`post` 结构体的字段有 `title` 和 `content`。它还有一个嵌套的匿名字段 `author`。该字段指定 `author` 组成了 `post` 结构体。现在 `post` 可以访问 `author` 结构体的所有字段和方法。我们同样给 `post` 结构体添加了 `details()` 方法，用于打印标题、内容和作者的全名与简介。

一旦结构体内嵌套了一个结构体字段，Go 可以使我们访问其嵌套的字段，好像这些字段属于外部结构体一样。所以上面第 11 行的 `p.author.fullName()` 可以替换为 `p.fullName()`。于是，`details()` 方法可以重写，如下所示：

```go
Copyfunc (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}
```

现在，我们的 `author` 和 `post` 结构体都已准备就绪，我们来创建一个博客帖子来完成这个程序。

```go
Copypackage main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post1.details()
}
```

在上面程序中，main 函数在第 31 行新建了一个 `author` 结构体变量。而在第 36 行，我们通过嵌套 `author1` 来创建一个 `post`。该程序输出：

```bash
CopyTitle:  Inheritance in Go  
Content:  Go supports composition instead of inheritance  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast
```

## 结构体切片的嵌套

我们可以进一步处理这个示例，使用博客帖子的切片来创建一个网站。

我们首先定义 `website` 结构体。请在上述代码里的 main 函数中，添加下面的代码，并运行它。

```go
Copytype website struct {  
        []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}
```

在你添加上述代码后，当你运行程序时，编译器将会报错，如下所示：

```bash
Copymain.go:31:9: syntax error: unexpected [, expecting field name or embedded type
```

这项错误指出了嵌套的结构体切片 `[]post`。错误的原因是结构体不能嵌套一个匿名切片。我们需要一个字段名。所以我们来修复这个错误，让编译器顺利通过。

```go
Copytype website struct {  
        posts []post
}
```

可以看到，我给帖子的切片 `[]post` 添加了字段名 `posts`。

现在我们来修改主函数，为我们的新网站创建一些帖子吧。

修改后的完整代码如下所示：

```go
Copypackage main

import (  
    "fmt"
)

type author struct {  
    firstName string
    lastName  string
    bio       string
}

func (a author) fullName() string {  
    return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {  
    title   string
    content string
    author
}

func (p post) details() {  
    fmt.Println("Title: ", p.title)
    fmt.Println("Content: ", p.content)
    fmt.Println("Author: ", p.fullName())
    fmt.Println("Bio: ", p.bio)
}

type website struct {  
 posts []post
}
func (w website) contents() {  
    fmt.Println("Contents of Website\n")
    for _, v := range w.posts {
        v.details()
        fmt.Println()
    }
}

func main() {  
    author1 := author{
        "Naveen",
        "Ramanathan",
        "Golang Enthusiast",
    }
    post1 := post{
        "Inheritance in Go",
        "Go supports composition instead of inheritance",
        author1,
    }
    post2 := post{
        "Struct instead of Classes in Go",
        "Go does not support classes but methods can be added to structs",
        author1,
    }
    post3 := post{
        "Concurrency",
        "Go is a concurrent language and not a parallel one",
        author1,
    }
    w := website{
        posts: []post{post1, post2, post3},
    }
    w.contents()
}
```

在上面的主函数中，我们创建了一个作者 `author1`，以及三个帖子 `post1`、`post2` 和 `post3`。我们最后通过嵌套三个帖子，在第 62 行创建了网站 `w`，并在下一行显示内容。

程序会输出：

```bash
CopyContents of Website

Title:  Inheritance in Go  
Content:  Go supports composition instead of inheritance  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast

Title:  Struct instead of Classes in Go  
Content:  Go does not support classes but methods can be added to structs  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast

Title:  Concurrency  
Content:  Go is a concurrent language and not a parallel one  
Author:  Naveen Ramanathan  
Bio:  Golang Enthusiast
```

# 28. 多态

Go 通过[接口]来实现多态。我们已经讨论过，在 Go 语言中，我们是隐式地实现接口。一个类型如果定义了接口所声明的全部[方法]，那它就实现了该接口。现在我们来看看，利用接口，Go 是如何实现多态的。

## 使用接口实现多态

一个类型如果定义了接口的所有方法，那它就隐式地实现了该接口。

**所有实现了接口的类型，都可以把它的值保存在一个接口类型的变量中。在 Go 中，我们使用接口的这种特性来实现多态**。

通过一个程序我们来理解 Go 语言的多态，它会计算一个组织机构的净收益。为了简单起见，我们假设这个虚构的组织所获得的收入来源于两个项目：`fixed billing` 和 `time and material`。该组织的净收益等于这两个项目的收入总和。同样为了简单起见，我们假设货币单位是美元，而无需处理美分。因此货币只需简单地用 `int` 来表示。

我们首先定义一个接口 `Income`。

```go
Copytype Income interface {  
    calculate() int
    source() string
}
```

上面定义了接口 `Interface`，它包含了两个方法：`calculate()` 计算并返回项目的收入，而 `source()` 返回项目名称。

下面我们定义一个表示 `FixedBilling` 项目的结构体类型。

```go
Copytype FixedBilling struct {  
    projectName string
    biddedAmount int
}
```

项目 `FixedBillin` 有两个字段：`projectName` 表示项目名称，而 `biddedAmount` 表示组织向该项目投标的金额。

`TimeAndMaterial` 结构体用于表示项目 Time and Material。

```go
Copytype TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}
```

结构体 `TimeAndMaterial` 拥有三个字段名：`projectName`、`noOfHours` 和 `hourlyRate`。

下一步我们给这些结构体类型定义方法，计算并返回实际收入和项目名称。

```go
Copyfunc (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}
```

在项目 `FixedBilling` 里面，收入就是项目的投标金额。因此我们返回 `FixedBilling` 类型的 `calculate()` 方法。

而在项目 `TimeAndMaterial` 里面，收入等于 `noOfHours` 和 `hourlyRate` 的乘积，作为 `TimeAndMaterial` 类型的 `calculate()` 方法的返回值。

我们还通过 `source()` 方法返回了表示收入来源的项目名称。

由于 `FixedBilling` 和 `TimeAndMaterial` 两个结构体都定义了 `Income` 接口的两个方法：`calculate()` 和 `source()`，因此这两个结构体都实现了 `Income` 接口。

我们来声明一个 `calculateNetIncome` 函数，用来计算并打印总收入。

```go
Copyfunc calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}
```

上面的[函数]接收一个 `Income` 接口类型的[切片]作为参数。该函数会遍历这个接口切片，并依个调用 `calculate()` 方法，计算出总收入。该函数同样也会通过调用 `source()` 显示收入来源。根据 `Income` 接口的具体类型，程序会调用不同的 `calculate()` 和 `source()` 方法。于是，我们在 `calculateNetIncome` 函数中就实现了多态。

如果在该组织以后增加了新的收入来源，`calculateNetIncome` 无需修改一行代码，就可以正确地计算总收入了。

最后就剩下这个程序的 `main` 函数了。

```go
Copyfunc main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    incomeStreams := []Income{project1, project2, project3}
    calculateNetIncome(incomeStreams)
}
```

在上面的 `main` 函数中，我们创建了三个项目，有两个是 `FixedBilling` 类型，一个是 `TimeAndMaterial` 类型。接着我们创建了一个 `Income` 类型的切片，存放了这三个项目。由于这三个项目都实现了 `Interface` 接口，因此可以把这三个项目放入 `Income` 切片。最后我们将该切片作为参数，调用了 `calculateNetIncome` 函数，显示了项目不同的收益和收入来源。

以下完整的代码供你参考。

```go
Copypackage main

import (  
    "fmt"
)

type Income interface {  
    calculate() int
    source() string
}

type FixedBilling struct {  
    projectName string
    biddedAmount int
}

type TimeAndMaterial struct {  
    projectName string
    noOfHours  int
    hourlyRate int
}

func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}

func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}

func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    incomeStreams := []Income{project1, project2, project3}
    calculateNetIncome(incomeStreams)
}
```

该程序会输出：

```
CopyIncome From Project 1 = $5000  
Income From Project 2 = $10000  
Income From Project 3 = $4000  
Net income of organisation = $19000
```

## 新增收益流

假设前面的组织通过广告业务，建立了一个新的收益流（Income Stream）。我们可以看到添加它非常简单，并且计算总收益也很容易，我们无需对 `calculateNetIncome` 函数进行任何修改。这就是多态的好处。

我们首先定义 `Advertisement` 类型，并在 `Advertisement` 类型中定义 `calculate()` 和 `source()` 方法。

```go
Copytype Advertisement struct {  
    adName     string
    CPC        int
    noOfClicks int
}

func (a Advertisement) calculate() int {  
    return a.CPC * a.noOfClicks
}

func (a Advertisement) source() string {  
    return a.adName
}
```

`Advertisement` 类型有三个字段，分别是 `adName`、`CPC`（每次点击成本）和 `noOfClicks`（点击次数）。广告的总收益等于 `CPC` 和 `noOfClicks` 的乘积。

现在我们稍微修改一下 `main` 函数，把新的收益流添加进来。

```go
Copyfunc main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
    popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
    incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
    calculateNetIncome(incomeStreams)
}
```

我们创建了两个广告项目，即 `bannerAd` 和 `popupAd`。`incomeStream` 切片包含了这两个创建的广告项目。

```go
Copypackage main

import (  
    "fmt"
)

type Income interface {  
    calculate() int
    source() string
}

type FixedBilling struct {  
    projectName  string
    biddedAmount int
}

type TimeAndMaterial struct {  
    projectName string
    noOfHours   int
    hourlyRate  int
}

type Advertisement struct {  
    adName     string
    CPC        int
    noOfClicks int
}

func (fb FixedBilling) calculate() int {  
    return fb.biddedAmount
}

func (fb FixedBilling) source() string {  
    return fb.projectName
}

func (tm TimeAndMaterial) calculate() int {  
    return tm.noOfHours * tm.hourlyRate
}

func (tm TimeAndMaterial) source() string {  
    return tm.projectName
}

func (a Advertisement) calculate() int {  
    return a.CPC * a.noOfClicks
}

func (a Advertisement) source() string {  
    return a.adName
}
func calculateNetIncome(ic []Income) {  
    var netincome int = 0
    for _, income := range ic {
        fmt.Printf("Income From %s = $%d\n", income.source(), income.calculate())
        netincome += income.calculate()
    }
    fmt.Printf("Net income of organisation = $%d", netincome)
}

func main() {  
    project1 := FixedBilling{projectName: "Project 1", biddedAmount: 5000}
    project2 := FixedBilling{projectName: "Project 2", biddedAmount: 10000}
    project3 := TimeAndMaterial{projectName: "Project 3", noOfHours: 160, hourlyRate: 25}
    bannerAd := Advertisement{adName: "Banner Ad", CPC: 2, noOfClicks: 500}
    popupAd := Advertisement{adName: "Popup Ad", CPC: 5, noOfClicks: 750}
    incomeStreams := []Income{project1, project2, project3, bannerAd, popupAd}
    calculateNetIncome(incomeStreams)
}
```

上面程序会输出：

```
CopyIncome From Project 1 = $5000  
Income From Project 2 = $10000  
Income From Project 3 = $4000  
Income From Banner Ad = $1000  
Income From Popup Ad = $3750  
Net income of organisation = $23750
```

你会发现，尽管我们新增了收益流，但却完全没有修改 `calculateNetIncome` 函数。这就是多态带来的好处。由于新的 `Advertisement` 同样实现了 `Income` 接口，所以我们能够向 `incomeStreams` 切片添加 `Advertisement`。`calculateNetIncome` 无需修改，因为它能够调用 `Advertisement` 类型的 `calculate()` 和 `source()` 方法。29. Defer

## 什么是 defer？

`defer` 语句的用途是：含有 `defer` 语句的函数，会在该函数将要返回之前，调用另一个函数。这个定义可能看起来很复杂，我们通过一个示例就很容易明白了。

## 示例

```go
Copypackage main

import (  
    "fmt"
)

func finished() {  
    fmt.Println("Finished finding largest")
}

func largest(nums []int) {  
    defer finished()
    fmt.Println("Started finding largest")
    max := nums[0]
    for _, v := range nums {
        if v > max {
            max = v
        }
    }
    fmt.Println("Largest number in", nums, "is", max)
}

func main() {  
    nums := []int{78, 109, 2, 563, 300}
    largest(nums)
}
```

上面的程序很简单，就是找出一个给定切片的最大值。`largest` 函数接收一个 int 类型的[切片](https://studygolang.com/articles/12121)作为参数，然后打印出该切片中的最大值。`largest` 函数的第一行的语句为 `defer finished()`。这表示在 `finished()` 函数将要返回之前，会调用 `finished()` 函数。运行该程序，你会看到有如下输出：

```
CopyStarted finding largest  
Largest number in [78 109 2 563 300] is 563  
Finished finding largest
```

`largest` 函数开始执行后，会打印上面的两行输出。而就在 `largest` 将要返回的时候，又调用了我们的延迟函数（Deferred Function），打印出 `Finished finding largest` 的文本。

## 延迟方法

`defer` 不仅限于[函数]的调用，调用[方法]也是合法的。我们写一个小程序来测试吧。

```go
Copypackage main

import (  
    "fmt"
)


type person struct {  
    firstName string
    lastName string
}

func (p person) fullName() {  
    fmt.Printf("%s %s",p.firstName,p.lastName)
}

func main() {  
    p := person {
        firstName: "John",
        lastName: "Smith",
    }
    defer p.fullName()
    fmt.Printf("Welcome ")  
}
```

在上面的例子中，我们在第 22 行延迟了一个方法调用。而其他的代码很直观，这里不再解释。该程序输出：

```
CopyWelcome John Smith
```

## 实参取值（Arguments Evaluation）

在 Go 语言中，并非在调用延迟函数的时候才确定实参，而是当执行 `defer` 语句的时候，就会对延迟函数的实参进行求值。

通过一个例子就能够理解了。

```go
Copypackage main

import (  
    "fmt"
)

func printA(a int) {  
    fmt.Println("value of a in deferred function", a)
}
func main() {  
    a := 5
    defer printA(a)
    a = 10
    fmt.Println("value of a before deferred function call", a)

}
```

在上面的程序里的第 11 行，`a` 的初始值为 5。在第 12 行执行 `defer` 语句的时候，由于 `a` 等于 5，因此延迟函数 `printA` 的实参也等于 5。接着我们在第 13 行将 `a` 的值修改为 10。下一行会打印出 `a` 的值。该程序输出：

```
Copyvalue of a before deferred function call 10  
value of a in deferred function 5
```

从上面的输出，我们可以看出，在调用了 `defer` 语句后，虽然我们将 `a` 修改为 10，但调用延迟函数 `printA(a)`后，仍然打印的是 5。

## defer 栈

当一个函数内多次调用 `defer` 时，Go 会把 `defer` 调用放入到一个栈中，随后按照后进先出（Last In First Out, LIFO）的顺序执行。

我们下面编写一个小程序，使用 `defer` 栈，将一个字符串逆序打印。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    name := "Naveen"
    fmt.Printf("Orignal String: %s\n", string(name))
    fmt.Printf("Reversed String: ")
    for _, v := range []rune(name) {
        defer fmt.Printf("%c", v)
    }
}
```

在上述程序中的第 11 行，`for range` 循环会遍历一个字符串，并在第 12 行调用了 `defer fmt.Printf("%c", v)`。这些延迟调用会添加到一个栈中，按照后进先出的顺序执行，因此，该字符串会逆序打印出来。该程序会输出：

```
CopyOrignal String: Naveen  
Reversed String: neevaN
```

## defer 的实际应用

目前为止，我们看到的代码示例，都没有体现出 `defer` 的实际用途。本节我们会看看 `defer` 的实际应用。

当一个函数应该在与当前代码流（Code Flow）无关的环境下调用时，可以使用 `defer`。我们通过一个用到了 [`WaitGroup`] 代码示例来理解这句话的含义。我们首先会写一个没有使用 `defer` 的程序，然后我们会用 `defer` 来修改，看到 `defer` 带来的好处。

```go
Copypackage main

import (  
    "fmt"
    "sync"
)

type rect struct {  
    length int
    width  int
}

func (r rect) area(wg *sync.WaitGroup) {  
    if r.length < 0 {
        fmt.Printf("rect %v's length should be greater than zero\n", r)
        wg.Done()
        return
    }
    if r.width < 0 {
        fmt.Printf("rect %v's width should be greater than zero\n", r)
        wg.Done()
        return
    }
    area := r.length * r.width
    fmt.Printf("rect %v's area %d\n", r, area)
    wg.Done()
}

func main() {  
    var wg sync.WaitGroup
    r1 := rect{-67, 89}
    r2 := rect{5, -67}
    r3 := rect{8, 9}
    rects := []rect{r1, r2, r3}
    for _, v := range rects {
        wg.Add(1)
        go v.area(&wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

在上面的程序里，我们在第 8 行创建了 `rect` 结构体，并在第 13 行创建了 `rect` 的方法 `area`，计算出矩形的面积。`area` 检查了矩形的长宽是否小于零。如果矩形的长宽小于零，它会打印出对应的提示信息，而如果大于零，它会打印出矩形的面积。

`main` 函数创建了 3 个 `rect` 类型的变量：`r1`、`r2` 和 `r3`。在第 34 行，我们把这 3 个变量添加到了 `rects` 切片里。该切片接着使用 `for range` 循环遍历，把 `area` 方法作为一个并发的 Go 协程进行调用（第 37 行）。我们用 `WaitGroup wg` 来确保 `main` 函数在其他协程执行完毕之后，才会结束执行。`WaitGroup` 作为参数传递给 `area` 方法后，在第 16 行、第 21 行和第 26 行通知 `main` 函数，表示现在协程已经完成所有任务。**如果你仔细观察，会发现 wg.Done() 只在 area 函数返回的时候才会调用。wg.Done() 应该在 area 将要返回之前调用，并且与代码流的路径（Path）无关，因此我们可以只调用一次 defer，来有效地替换掉 wg.Done() 的多次调用**。

我们来用 `defer` 来重写上面的代码。

在下面的代码中，我们移除了原先程序中的 3 个 `wg.Done` 的调用，而是用一个单独的 `defer wg.Done()` 来取代它（第 14 行）。这使得我们的代码更加简洁易懂。

```go
Copypackage main

import (  
    "fmt"
    "sync"
)

type rect struct {  
    length int
    width  int
}

func (r rect) area(wg *sync.WaitGroup) {  
    defer wg.Done()
    if r.length < 0 {
        fmt.Printf("rect %v's length should be greater than zero\n", r)
        return
    }
    if r.width < 0 {
        fmt.Printf("rect %v's width should be greater than zero\n", r)
        return
    }
    area := r.length * r.width
    fmt.Printf("rect %v's area %d\n", r, area)
}

func main() {  
    var wg sync.WaitGroup
    r1 := rect{-67, 89}
    r2 := rect{5, -67}
    r3 := rect{8, 9}
    rects := []rect{r1, r2, r3}
    for _, v := range rects {
        wg.Add(1)
        go v.area(&wg)
    }
    wg.Wait()
    fmt.Println("All go routines finished executing")
}
```

该程序会输出：

```
Copyrect {8 9}'s area 72  
rect {-67 89}'s length should be greater than zero  
rect {5 -67}'s width should be greater than zero  
All go routines finished executing
```

在上面的程序中，使用 `defer` 还有一个好处。假设我们使用 `if` 条件语句，又给 `area` 方法添加了一条返回路径（Return Path）。如果没有使用 `defer` 来调用 `wg.Done()`，我们就得很小心了，确保在这条新添的返回路径里调用了 `wg.Done()`。由于现在我们延迟调用了 `wg.Done()`，因此无需再为这条新的返回路径添加 `wg.Done()` 了。

# 30. 错误处理

## 什么是错误？

错误表示程序中出现了异常情况。比如当我们试图打开一个文件时，文件系统里却并没有这个文件。这就是异常情况，它用一个错误来表示。

在 Go 中，错误一直是很常见的。错误用内建的 `error` 类型来表示。

就像其他的内建类型（如 `int`、`float64` 等），错误值可以存储在变量里、作为函数的返回值等等。

## 示例

现在我们开始编写一个示例，该程序试图打开一个并不存在的文件。

```go
Copypackage main

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

在程序的第 9 行，我们试图打开路径为 `/test.txt` 的文件。`os` 包里的 [`Open`]函数有如下签名：

```go
Copyfunc Open(name string) (file *File, err error)
```

**如果成功打开文件，Open 函数会返回一个文件句柄（File Handler）和一个值为 nil 的错误。而如果打开文件时发生了错误，会返回一个不等于 nil 的错误**。

如果一个[函数] 或[方法] 返回了错误，按照惯例，错误会作为最后一个值返回。于是 `Open` 函数也是将 `err` 作为最后一个返回值。

**按照 Go 的惯例，在处理错误时，通常都是将返回的错误与 nil 比较。nil 值表示了没有错误发生，而非 nil 值表示出现了错误**。在这里，我们第 10 行检查了错误值是否为 `nil`。如果不是 `nil`，我们会简单地打印出错误，并在 `main` 函数中返回。

运行该程序会输出：

```
Copyopen /test.txt: No such file or directory
```

很棒！我们得到了一个错误，它指出该文件并不存在。

## 错误类型的表示

让我们进一步深入，理解 `error` 类型是如何定义的。`error` 是一个[接口]类型，定义如下：

```go
Copytype error interface {  
    Error() string
}
```

`error` 有了一个签名为 `Error() string` 的方法。所有实现该接口的类型都可以当作一个错误类型。`Error()` 方法给出了错误的描述。

`fmt.Println` 在打印错误时，会在内部调用 `Error() string` 方法来得到该错误的描述。上一节示例中的第 11 行，就是这样打印出错误的描述的。

## 从错误获取更多信息的不同方法

现在，我们知道了 `error` 是一个接口类型，让我们看看如何从一个错误获取更多信息。

在前面的示例里，我们只是打印出错误的描述。如果我们想知道这个错误的文件路径，该怎么做呢？一种选择是直接解析错误的字符串。这是前面示例的输出：

```
Copyopen /test.txt: No such file or directory
```

**我们解析了这条错误信息，虽然获取了发生错误的文件路径，但是这种方法很不优雅。随着语言版本的更新，这条错误的描述随时都有可能变化，使我们程序出错**。

有没有更加可靠的方法来获取文件名呢？答案是肯定的，这是可以做到的，Go 标准库给出了各种提取错误相关信息的方法。我们一个个来看看吧。

### 1. 断言底层结构体类型，使用结构体字段获取更多信息

如果你仔细阅读了 [`Open`] 函数的文档，你可以看见它返回的错误类型是 `*PathError`。[`PathError`]是[结构体]类型，它在标准库中的实现如下：

```go
Copytype PathError struct {  
    Op   string
    Path string
    Err  error
}

func (e *PathError) Error() string { return e.Op + " " + e.Path + ": " + e.Err.Error() }
```

通过上面的代码，你就知道了 `*PathError` 通过声明 `Error() string` 方法，实现了 `error` 接口。`Error() string` 将文件操作、路径和实际错误拼接，并返回该字符串。于是我们得到该错误信息：

```
Copyopen /test.txt: No such file or directory
```

结构体 `PathError` 的 `Path` 字段，就有导致错误的文件路径。我们修改前面写的程序，打印出该路径。

```go
Copypackage main

import (  
    "fmt"
    "os"
)

func main() {  
    f, err := os.Open("/test.txt")
    if err, ok := err.(*os.PathError); ok {
        fmt.Println("File at path", err.Path, "failed to open")
        return
    }
    fmt.Println(f.Name(), "opened successfully")
}
```

在上面的程序里，我们在第 10 行使用了[类型断言]（Type Assertion）来获取 `error` 接口的底层值（Underlying Value）。接下来在第 11 行，我们使用 `err.Path` 来打印该路径。该程序会输出：

```
CopyFile at path /test.txt failed to open
```

很棒！我们已经使用类型断言成功获取到了该错误的文件路径。

### 2. 断言底层结构体类型，调用方法获取更多信息

第二种获取更多错误信息的方法，也是对底层类型进行断言，然后通过调用该结构体类型的方法，来获取更多的信息。

我们通过一个实例来理解这一点。

标准库中的 `DNSError` 结构体类型定义如下：

```go
Copytype DNSError struct {  
    ...
}

func (e *DNSError) Error() string {  
    ...
}
func (e *DNSError) Timeout() bool {  
    ... 
}
func (e *DNSError) Temporary() bool {  
    ... 
}
```

从上述代码可以看到，`DNSError` 结构体还有 `Timeout() bool` 和 `Temporary() bool` 两个方法，它们返回一个布尔值，指出该错误是由超时引起的，还是临时性错误。

接下来我们编写一个程序，断言 `*DNSError` 类型，并调用这些方法来确定该错误是临时性错误，还是由超时导致的。

```go
Copypackage main

import (  
    "fmt"
    "net"
)

func main() {  
    addr, err := net.LookupHost("golangbot123.com")
    if err, ok := err.(*net.DNSError); ok {
        if err.Timeout() {
            fmt.Println("operation timed out")
        } else if err.Temporary() {
            fmt.Println("temporary error")
        } else {
            fmt.Println("generic error: ", err)
        }
        return
    }
    fmt.Println(addr)
}
```

在上述程序中，我们在第 9 行，试图获取 `golangbot123.com`（无效的域名） 的 ip。在第 10 行，我们通过 `*net.DNSError` 的类型断言，获取到了错误的底层值。接下来的第 11 行和第 13 行，我们分别检查了该错误是由超时引起的，还是一个临时性错误。

在本例中，我们的错误既不是临时性错误，也不是由超时引起的，因此该程序输出：

```
Copygeneric error:  lookup golangbot123.com: no such host
```

如果该错误是临时性错误，或是由超时引发的，那么对应的 if 语句会执行，于是我们就可以适当地处理它们。

### 3. 直接比较

第三种获取错误的更多信息的方式，是与 `error` 类型的变量直接比较。我们通过一个示例来理解。

`filepath` 包中的 [`Glob`] 用于返回满足 glob 模式的所有文件名。如果模式写的不对，该函数会返回一个错误 `ErrBadPattern`。

`filepath` 包中的 `ErrBadPattern` 定义如下：

```go
Copyvar ErrBadPattern = errors.New("syntax error in pattern")
```

`errors.New()` 用于创建一个新的错误。我们会在下一教程中详细讨论它。

当模式不正确时，`Glob` 函数会返回 `ErrBadPattern`。

我们来写一个小程序来看看这个错误。

```go
Copypackage main

import (  
    "fmt"
    "path/filepath"
)

func main() {  
    files, error := filepath.Glob("[")
    if error != nil && error == filepath.ErrBadPattern {
        fmt.Println(error)
        return
    }
    fmt.Println("matched files", files)
}
```

在上述程序里，我们查询了模式为 `[` 的文件，然而这个模式写的不正确。我们检查了该错误是否为 `nil`。为了获取该错误的更多信息，我们在第 10 行将 `error` 直接与 `filepath.ErrBadPattern` 相比较。如果该条件满足，那么该错误就是由模式错误导致的。该程序会输出：

```
Copysyntax error in pattern
```

标准库在提供错误的详细信息时，使用到了上述提到的三种方法。在下一教程里，我们会通过这些方法来创建我们自己的自定义错误。

## 不可忽略错误

绝不要忽略错误。忽视错误会带来问题。接下来我重写上面的示例，在列出所有满足模式的文件名时，我省略了错误处理的代码。

```go
Copypackage main

import (  
    "fmt"
    "path/filepath"
)

func main() {  
    files, _ := filepath.Glob("[")
    fmt.Println("matched files", files)
}
```

我们已经从前面的示例知道了这个模式是错误的。在第 9 行，通过使用 `_` 空白标识符，我忽略了 `Glob` 函数返回的错误。我在第 10 行简单打印了所有匹配的文件。该程序会输出：

```
Copymatched files []
```

由于我忽略了错误，输出看起来就像是没有任何匹配了 glob 模式的文件，但实际上这是因为模式的写法不对。所以绝不要忽略错误。

本教程到此结束。

这一教程我们讨论了该如何处理程序中出现的错误，也讨论了如何查询关于错误的更多信息。简单概括一下本教程讨论的内容：

- 什么是错误？
- 错误的表示
- 获取错误详细信息的各种方法
- 不能忽视错误

# 31. 自定义错误

## 使用 New 函数创建自定义错误

创建自定义错误最简单的方法是使用 [`errors`]包中的 [`New`]函数。

在使用 New [函数]创建自定义错误之前，我们先来看看 `New` 是如何实现的。如下所示，是 [`errors` 包]中的 `New` 函数的实现。

```go
Copy// Package errors implements functions to manipulate errors.
package errors

// New returns an error that formats as the given text.
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

`New` 函数的实现很简单。`errorString` 是一个[结构体]类型，只有一个字符串字段 `s`。第 14 行使用了 `errorString` 指针接受者（Pointer Receiver），来实现 `error` 接口的 `Error() string` [方法]。

第 5 行的 `New` 函数有一个字符串参数，通过这个参数创建了 `errorString` 类型的变量，并返回了它的地址。于是它就创建并返回了一个新的错误。

现在我们已经知道了 `New` 函数是如何工作的，我们开始在程序里使用 `New` 来创建自定义错误吧。

我们将创建一个计算圆半径的简单程序，如果半径为负，它会返回一个错误。

```go
Copypackage main

import (  
    "errors"
    "fmt"
    "math"
)

func circleArea(radius float64) (float64, error) {  
    if radius < 0 {
        return 0, errors.New("Area calculation failed, radius is less than zero")
    }
    return math.Pi * radius * radius, nil
}

func main() {  
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of circle %0.2f", area)
}
```

在上面的程序中，我们检查半径是否小于零（第 10 行）。如果半径小于零，我们会返回等于 0 的面积，以及相应的错误信息。如果半径大于零，则会计算出面积，并返回值为 `nil` 的错误（第 13 行）。

在 `main` 函数里，我们在第 19 行检查错误是否等于 `nil`。如果不是 `nil`，我们会打印出错误并返回，否则我们会打印出圆的面积。

在我们的程序中，半径小于零，因此打印出：

```
CopyArea calculation failed, radius is less than zero
```

## 使用 Errorf 给错误添加更多信息

上面的程序效果不错，但是如果我们能够打印出当前圆的半径，那就更好了。这就要用到 [`fmt`]包中的 [`Errorf`] 函数了。`Errorf` 函数会根据格式说明符，规定错误的格式，并返回一个符合该错误的[字符串]。

接下来我们使用 `Errorf` 函数来改进我们的程序。

```go
Copypackage main

import (  
    "fmt"
    "math"
)

func circleArea(radius float64) (float64, error) {  
    if radius < 0 {
        return 0, fmt.Errorf("Area calculation failed, radius %0.2f is less than zero", radius)
    }
    return math.Pi * radius * radius, nil
}

func main() {  
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of circle %0.2f", area)
}
```

在上面的程序中，我们使用 `Errorf`（第 10 行）打印了发生错误的半径。程序运行后会输出：

```
CopyArea calculation failed, radius -20.00 is less than zero
```

## 使用结构体类型和字段提供错误的更多信息

错误还可以用实现了 `error` [接口]的结构体来表示。这种方式可以更加灵活地处理错误。在上面例子中，如果我们希望访问引发错误的半径，现在唯一的方法就是解析错误的描述信息 `Area calculation failed, radius -20.00 is less than zero`。这样做不太好，因为一旦描述信息发生变化，程序就会出错。

我们会使用标准库里采用的方法，在上一教程中“断言底层结构体类型，使用结构体字段获取更多信息”这一节，我们讲解了这一方法，可以使用结构体字段来访问引发错误的半径。我们会创建一个实现 `error` 接口的结构体类型，并使用它的字段来提供关于错误的更多信息。

第一步就是创建一个表示错误的结构体类型。错误类型的命名约定是名称以 `Error` 结尾。因此我们不妨把结构体类型命名为 `areaError`。

```go
Copytype areaError struct {  
    err    string
    radius float64
}
```

上面的结构体类型有一个 `radius` 字段，它存储了与错误有关的半径，而 `err` 字段存储了实际的错误信息。

下一步是实现 `error` 接口。

```go
Copyfunc (e *areaError) Error() string {  
    return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}
```

在上面的代码中，我们使用指针接收者 `*areaError`，实现了 `error` 接口的 `Error() string` 方法。该方法打印出半径和关于错误的描述。

现在我们来编写 `main` 函数和 `circleArea` 函数来完成整个程序。

```go
Copypackage main

import (  
    "fmt"
    "math"
)

type areaError struct {  
    err    string
    radius float64
}

func (e *areaError) Error() string {  
    return fmt.Sprintf("radius %0.2f: %s", e.radius, e.err)
}

func circleArea(radius float64) (float64, error) {  
    if radius < 0 {
        return 0, &areaError{"radius is negative", radius}
    }
    return math.Pi * radius * radius, nil
}

func main() {  
    radius := -20.0
    area, err := circleArea(radius)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            fmt.Printf("Radius %0.2f is less than zero", err.radius)
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Printf("Area of rectangle1 %0.2f", area)
}
```

[在 playground 上运行](https://play.golang.org/p/OTs7J0adQg)

在上面的程序中，`circleArea`（第 17 行）用于计算圆的面积。该函数首先检查半径是否小于零，如果小于零，它会通过错误半径和对应错误信息，创建一个 `areaError` 类型的值，然后返回 `areaError` 值的地址，与此同时 `area` 等于 0（第 19 行）。**于是我们提供了更多的错误信息（即导致错误的半径），我们使用了自定义错误的结构体字段来定义它**。

如果半径是非负数，该函数会在第 21 行计算并返回面积，同时错误值为 `nil`。

在 `main` 函数的 26 行，我们试图计算半径为 -20 的圆的面积。由于半径小于零，因此会导致一个错误。

我们在第 27 行检查了错误是否为 `nil`，并在下一行断言了 `*areaError` 类型。**如果错误是 \*areaError 类型，我们就可以用 err.radius 来获取错误的半径（第 29 行），打印出自定义错误的消息，最后程序返回退出**。

如果断言错误，我们就在第 32 行打印该错误，并返回。如果没有发生错误，在第 35 行会打印出面积。

该程序会输出：

```
CopyRadius -20.00 is less than zero
```

下面我们来使用上一教程提到的[第二种方法]，使用自定义错误类型的方法来提供错误的更多信息。

## 使用结构体类型的方法来提供错误的更多信息

在本节里，我们会编写一个计算矩形面积的程序。如果长或宽小于零，程序就会打印出错误。

第一步就是创建一个表示错误的结构体。

```go
Copytype areaError struct {  
    err    string //error description
    length float64 //length which caused the error
    width  float64 //width which caused the error
}
```

上面的结构体类型除了有一个错误描述字段，还有可能引发错误的宽和高。

现在我们有了错误类型，我们来实现 `error` 接口，并给该错误类型添加两个方法，使它提供了更多的错误信息。

```go
Copyfunc (e *areaError) Error() string {  
    return e.err
}

func (e *areaError) lengthNegative() bool {  
    return e.length < 0
}

func (e *areaError) widthNegative() bool {  
    return e.width < 0
}
```

在上面的代码片段中，我们从 `Error() string` 方法中返回了关于错误的描述。当 `length` 小于零时，`lengthNegative() bool` 方法返回 `true`，而当 `width` 小于零时，`widthNegative() bool` 方法返回 `true`。**这两个方法都提供了关于错误的更多信息，在这里，它提示我们计算面积失败的原因（长度为负数或者宽度为负数）。于是我们就有了两个错误类型结构体的方法，来提供更多的错误信息**。

下一步就是编写计算面积的函数。

```go
Copyfunc rectArea(length, width float64) (float64, error) {  
    err := ""
    if length < 0 {
        err += "length is less than zero"
    }
    if width < 0 {
        if err == "" {
            err = "width is less than zero"
        } else {
            err += ", width is less than zero"
        }
    }
    if err != "" {
        return 0, &areaError{err, length, width}
    }
    return length * width, nil
}
```

上面的 `rectArea` 函数检查了长或宽是否小于零，如果小于零，`rectArea` 会返回一个错误信息，否则 `rectArea` 会返回矩形的面积和一个值为 `nil` 的错误。

让我们创建 `main` 函数来完成整个程序。

```go
Copyfunc main() {  
    length, width := -5.0, -9.0
    area, err := rectArea(length, width)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            if err.lengthNegative() {
                fmt.Printf("error: length %0.2f is less than zero\n", err.length)

            }
            if err.widthNegative() {
                fmt.Printf("error: width %0.2f is less than zero\n", err.width)

            }
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Println("area of rect", area)
}
```

在 `main` 程序中，我们检查了错误是否为 `nil`（第 4 行）。如果错误值不是 `nil`，我们会在下一行断言 `*areaError` 类型。然后，我们使用 `lengthNegative()` 和 `widthNegative()` 方法，检查错误的原因是长度小于零还是宽度小于零。这样我们就使用了错误结构体类型的方法，来提供更多的错误信息。

如果没有错误发生，就会打印矩形的面积。

下面是整个程序的代码供你参考。

```go
Copypackage main

import "fmt"

type areaError struct {  
    err    string  //error description
    length float64 //length which caused the error
    width  float64 //width which caused the error
}

func (e *areaError) Error() string {  
    return e.err
}

func (e *areaError) lengthNegative() bool {  
    return e.length < 0
}

func (e *areaError) widthNegative() bool {  
    return e.width < 0
}

func rectArea(length, width float64) (float64, error) {  
    err := ""
    if length < 0 {
        err += "length is less than zero"
    }
    if width < 0 {
        if err == "" {
            err = "width is less than zero"
        } else {
            err += ", width is less than zero"
        }
    }
    if err != "" {
        return 0, &areaError{err, length, width}
    }
    return length * width, nil
}

func main() {  
    length, width := -5.0, -9.0
    area, err := rectArea(length, width)
    if err != nil {
        if err, ok := err.(*areaError); ok {
            if err.lengthNegative() {
                fmt.Printf("error: length %0.2f is less than zero\n", err.length)

            }
            if err.widthNegative() {
                fmt.Printf("error: width %0.2f is less than zero\n", err.width)

            }
            return
        }
        fmt.Println(err)
        return
    }
    fmt.Println("area of rect", area)
}
```

该程序会打印输出：

```
Copyerror: length -5.00 is less than zero  
error: width -9.00 is less than zero
```

在上一教程[错误处理]中，我们介绍了三种提供更多错误信息的方法，现在我们已经看了其中两个示例。

第三种方法使用的是直接比较，比较简单。我留给读者作为练习，你们可以试着使用这种方法来给出自定义错误的更多信息。

本教程到此结束。

简单概括一下本教程讨论的内容：

- 使用 `New` 函数创建自定义错误
- 使用 `Error` 添加更多错误信息
- 使用结构体类型和字段，提供更多错误信息
- 使用结构体类型和方法，提供更多错误信息

# 32. panic 和 recover

## 什么是 panic？

在 Go 语言中，程序中一般是使用[错误]来处理异常情况。对于程序中出现的大部分异常情况，错误就已经够用了。

但在有些情况，当程序发生异常时，无法继续运行。在这种情况下，我们会使用 `panic` 来终止程序。当[函数]发生 panic 时，它会终止运行，在执行完所有的[延迟]函数后，程序控制返回到该函数的调用方。这样的过程会一直持续下去，直到当前[协程]的所有函数都返回退出，然后程序会打印出 panic 信息，接着打印出堆栈跟踪（Stack Trace），最后程序终止。在编写一个示例程序后，我们就能很好地理解这个概念了。

在本教程里，我们还会接着讨论，当程序发生 panic 时，使用 `recover` 可以重新获得对该程序的控制。

可以认为 `panic` 和 `recover` 与其他语言中的 `try-catch-finally` 语句类似，只不过一般我们很少使用 `panic` 和 `recover`。而当我们使用了 `panic` 和 `recover` 时，也会比 `try-catch-finally` 更加优雅，代码更加整洁。

## 什么时候应该使用 panic？

**需要注意的是，你应该尽可能地使用错误，而不是使用 panic 和 recover。只有当程序不能继续运行的时候，才应该使用 panic 和 recover 机制**。

panic 有两个合理的用例。

1. **发生了一个不能恢复的错误，此时程序不能继续运行**。 一个例子就是 web 服务器无法绑定所要求的端口。在这种情况下，就应该使用 panic，因为如果不能绑定端口，啥也做不了。
2. **发生了一个编程上的错误**。 假如我们有一个接收指针参数的方法，而其他人使用 `nil` 作为参数调用了它。在这种情况下，我们可以使用 panic，因为这是一个编程错误：用 `nil` 参数调用了一个只能接收合法指针的方法。

## panic 示例

内建函数 `panic` 的签名如下所示：

```go
Copyfunc panic(interface{})
```

当程序终止时，会打印传入 `panic` 的参数。我们写一个示例，你就会清楚它的用途了。我们现在就开始吧。

我们会写一个例子，来展示 `panic` 如何工作。

```go
Copypackage main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```

上面的程序很简单，会打印一个人的全名。第 7 行的 `fullName` 函数会打印出一个人的全名。该函数在第 8 行和第 11 行分别检查了 `firstName` 和 `lastName` 的指针是否为 `nil`。如果是 `nil`，`fullName` 函数会调用含有不同的错误信息的 `panic`。当程序终止时，会打印出该错误信息。

运行该程序，会有如下输出：

```
Copypanic: runtime error: last name cannot be nil

goroutine 1 [running]:  
main.fullName(0x1040c128, 0x0)  
    /tmp/sandbox135038844/main.go:12 +0x120
main.main()  
    /tmp/sandbox135038844/main.go:20 +0x80
```

我们来分析这个输出，理解一下 panic 是如何工作的，并且思考当程序发生 panic 时，会怎样打印堆栈跟踪。

在第 19 行，我们将 `Elon` 赋值给了 `firstName`。在第 20 行，我们调用了 `fullName` 函数，其中 `lastName` 等于 `nil`。因此，满足了第 11 行的条件，程序发生 panic。当出现了 panic 时，程序就会终止运行，打印出传入 panic 的参数，接着打印出堆栈跟踪。因此，第 14 行和第 15 行的代码并不会在发生 panic 之后执行。程序首先会打印出传入 `panic` 函数的信息：

```
Copypanic: runtime error: last name cannot be empty
```

接着打印出堆栈跟踪。

程序在 `fullName` 函数的第 12 行发生 panic，因此，首先会打印出如下所示的输出。

```
Copymain.fullName(0x1040c128, 0x0)  
    /tmp/sandbox135038844/main.go:12 +0x120
```

接着会打印出堆栈的下一项。在本例中，堆栈跟踪中的下一项是第 20 行（因为发生 panic 的 `fullName` 调用就在这一行），因此接下来会打印出：

```
Copymain.main()  
    /tmp/sandbox135038844/main.go:20 +0x80
```

现在我们已经到达了导致 panic 的顶层函数，这里没有更多的层级，因此结束打印。

## 发生 panic 时的 defer

我们重新总结一下 panic 做了什么。**当函数发生 panic 时，它会终止运行，在执行完所有的延迟函数后，程序控制返回到该函数的调用方。这样的过程会一直持续下去，直到当前协程的所有函数都返回退出，然后程序会打印出 panic 信息，接着打印出堆栈跟踪，最后程序终止**。

在上面的例子中，我们没有延迟调用任何函数。如果有延迟函数，会先调用它，然后程序控制返回到函数调用方。

我们来修改上面的示例，使用一个延迟语句。

```go
Copypackage main

import (  
    "fmt"
)

func fullName(firstName *string, lastName *string) {  
    defer fmt.Println("deferred call in fullName")
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```

上述代码中，我们只修改了两处，分别在第 8 行和第 20 行添加了延迟函数的调用。

该函数会打印：

```
CopyThis program prints,

deferred call in fullName  
deferred call in main  
panic: runtime error: last name cannot be nil

goroutine 1 [running]:  
main.fullName(0x1042bf90, 0x0)  
    /tmp/sandbox060731990/main.go:13 +0x280
main.main()  
    /tmp/sandbox060731990/main.go:22 +0xc0
```

当程序在第 13 行发生 panic 时，首先执行了延迟函数，接着控制返回到函数调用方，调用方的延迟函数继续运行，直到到达顶层调用函数。

在我们的例子中，首先执行 `fullName` 函数中的 `defer` 语句（第 8 行）。程序打印出：

```
Copydeferred call in fullName
```

接着程序返回到 `main` 函数，执行了 `main` 函数的延迟调用，因此会输出：

```
Copydeferred call in main
```

现在程序控制到达了顶层函数，因此该函数会打印出 panic 信息，然后是堆栈跟踪，最后终止程序。

## recover

`recover` 是一个内建函数，用于重新获得 panic 协程的控制。

`recover` 函数的标签如下所示：

```go
Copyfunc recover() interface{}
```

只有在延迟函数的内部，调用 `recover` 才有用。在延迟函数内调用 `recover`，可以取到 `panic` 的错误信息，并且停止 panic 续发事件（Panicking Sequence），程序运行恢复正常。如果在延迟函数的外部调用 `recover`，就不能停止 panic 续发事件。

我们来修改一下程序，在发生 panic 之后，使用 `recover` 来恢复正常的运行。

```go
Copypackage main

import (  
    "fmt"
)

func recoverName() {  
    if r := recover(); r!= nil {
        fmt.Println("recovered from ", r)
    }
}

func fullName(firstName *string, lastName *string) {  
    defer recoverName()
    if firstName == nil {
        panic("runtime error: first name cannot be nil")
    }
    if lastName == nil {
        panic("runtime error: last name cannot be nil")
    }
    fmt.Printf("%s %s\n", *firstName, *lastName)
    fmt.Println("returned normally from fullName")
}

func main() {  
    defer fmt.Println("deferred call in main")
    firstName := "Elon"
    fullName(&firstName, nil)
    fmt.Println("returned normally from main")
}
```

在第 7 行，`recoverName()` 函数调用了 `recover()`，返回了调用 `panic` 的传参。在这里，我们只是打印出 `recover` 的返回值（第 8 行）。在 `fullName` 函数内，我们在第 14 行延迟调用了 `recoverNames()`。

当 `fullName` 发生 panic 时，会调用延迟函数 `recoverName()`，它使用了 `recover()` 来停止 panic 续发事件。

该程序会输出：

```
Copyrecovered from  runtime error: last name cannot be nil  
returned normally from main  
deferred call in main
```

当程序在第 19 行发生 panic 时，会调用延迟函数 `recoverName`，它反过来会调用 `recover()` 来重新获得 panic 协程的控制。第 8 行调用了 `recover`，返回了 `panic` 的传参，因此会打印：

```
Copyrecovered from  runtime error: last name cannot be nil
```

在执行完 `recover()` 之后，panic 会停止，程序控制返回到调用方（在这里就是 `main` 函数），程序在发生 panic 之后，从第 29 行开始会继续正常地运行。程序会打印 `returned normally from main`，之后是 `deferred call in main`。

## panic，recover 和 Go 协程

只有在相同的 [Go 协程]中调用 recover 才管用。`recover` 不能恢复一个不同协程的 panic。我们用一个例子来理解这一点。

```go
Copypackage main

import (  
    "fmt"
    "time"
)

func recovery() {  
    if r := recover(); r != nil {
        fmt.Println("recovered:", r)
    }
}

func a() {  
    defer recovery()
    fmt.Println("Inside A")
    go b()
    time.Sleep(1 * time.Second)
}

func b() {  
    fmt.Println("Inside B")
    panic("oh! B panicked")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}
```

在上面的程序中，函数 `b()` 在第 23 行发生 panic。函数 `a()` 调用了一个延迟函数 `recovery()`，用于恢复 panic。在第 17 行，函数 `b()` 作为一个不同的协程来调用。下一行的 `Sleep` 只是保证 `a()` 在 `b()` 运行结束之后才退出。

你认为程序会输出什么？panic 能够恢复吗？答案是否定的，panic 并不会恢复。因为调用 `recovery` 的协程和 `b()` 中发生 panic 的协程并不相同，因此不可能恢复 panic。

运行该程序会输出：

```
CopyInside A  
Inside B  
panic: oh! B panicked

goroutine 5 [running]:  
main.b()  
    /tmp/sandbox388039916/main.go:23 +0x80
created by main.a  
    /tmp/sandbox388039916/main.go:17 +0xc0
```

从输出可以看出，panic 没有恢复。

如果函数 `b()` 在相同的协程里调用，panic 就可以恢复。

如果程序的第 17 行由 `go b()` 修改为 `b()`，就可以恢复 panic 了，因为 panic 发生在与 recover 相同的协程里。如果运行这个修改后的程序，会输出：

```
CopyInside A  
Inside B  
recovered: oh! B panicked  
normally returned from main
```

## 运行时 panic

运行时错误（如数组越界）也会导致 panic。这等价于调用了内置函数 `panic`，其参数由接口类型 [runtime.Error]给出。`runtime.Error` 接口的定义如下：

```go
Copytype Error interface {  
    error
    // RuntimeError is a no-op function but
    // serves to distinguish types that are run time
    // errors from ordinary errors: a type is a
    // run time error if it has a RuntimeError method.
    RuntimeError()
}
```

而 `runtime.Error` 接口满足内建接口类型 [`error`]。

我们来编写一个示例，创建一个运行时 panic。

```go
Copypackage main

import (  
    "fmt"
)

func a() {  
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}
func main() {  
    a()
    fmt.Println("normally returned from main")
}
```

在上面的程序中，第 9 行我们试图访问 `n[3]`，这是一个对[切片]的错误引用。该程序会发生 panic，输出如下：

```
Copypanic: runtime error: index out of range

goroutine 1 [running]:  
main.a()  
    /tmp/sandbox780439659/main.go:9 +0x40
main.main()  
    /tmp/sandbox780439659/main.go:13 +0x20
```

你也许想知道，是否可以恢复一个运行时 panic？当然可以！我们来修改一下上面的代码，恢复这个 panic。

```go
Copypackage main

import (  
    "fmt"
)

func r() {  
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
    }
}

func a() {  
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}
```

运行上面程序会输出：

```
CopyRecovered runtime error: index out of range  
normally returned from main
```

从输出可以知道，我们已经恢复了这个 panic。

## 恢复后获得堆栈跟踪

当我们恢复 panic 时，我们就释放了它的堆栈跟踪。实际上，在上述程序里，恢复 panic 之后，我们就失去了堆栈跟踪。

有办法可以打印出堆栈跟踪，就是使用 [`Debug`]包中的 [`PrintStack`]函数。

```go
Copypackage main

import (  
    "fmt"
    "runtime/debug"
)

func r() {  
    if r := recover(); r != nil {
        fmt.Println("Recovered", r)
        debug.PrintStack()
    }
}

func a() {  
    defer r()
    n := []int{5, 7, 4}
    fmt.Println(n[3])
    fmt.Println("normally returned from a")
}

func main() {  
    a()
    fmt.Println("normally returned from main")
}
```

在上面的程序中，我们在第 11 行使用了 `debug.PrintStack()` 打印堆栈跟踪。

该程序会输出：

```
CopyRecovered runtime error: index out of range  
goroutine 1 [running]:  
runtime/debug.Stack(0x1042beb8, 0x2, 0x2, 0x1c)  
    /usr/local/go/src/runtime/debug/stack.go:24 +0xc0
runtime/debug.PrintStack()  
    /usr/local/go/src/runtime/debug/stack.go:16 +0x20
main.r()  
    /tmp/sandbox949178097/main.go:11 +0xe0
panic(0xf0a80, 0x17cd50)  
    /usr/local/go/src/runtime/panic.go:491 +0x2c0
main.a()  
    /tmp/sandbox949178097/main.go:18 +0x80
main.main()  
    /tmp/sandbox949178097/main.go:23 +0x20
normally returned from main
```

从输出我们可以看出，首先已经恢复了 panic，打印出 `Recovered runtime error: index out of range`。此外，我们也打印出了堆栈跟踪。在恢复了 panic 之后，还打印出 `normally returned from main`。

本教程到此结束。

简单概括一下本教程讨论的内容：

- 什么是 panic？
- 什么时候应该使用 panic？
- panic 示例
- 发生 panic 时的 defer
- recover
- panic，recover 和 Go 协程
- 运行时 panic
- 恢复后获得堆栈跟踪



# 33. 函数是一等公民（头等函数）

现在简单概括一下本教程讨论的内容：

- 什么是头等函数？
- 匿名函数
- 用户自定义的函数类型
- 高阶函数
  - 把函数作为参数，传递给其它函数
  - 在其它函数中返回函数
- 闭包
- 头等函数的实际用途

## 什么是头等函数？

**支持头等函数（First Class Function）的编程语言，可以把函数赋值给变量，也可以把函数作为其它函数的参数或者返回值。Go 语言支持头等函数的机制**。

本教程我们会讨论头等函数的语法和用例。

## 匿名函数

我们来编写一个简单的示例，把函数赋值给一个变量。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    a := func() {
        fmt.Println("hello world first class function")
    }
    a()
    fmt.Printf("%T", a)
}
```

在上面的程序中，我们将一个函数赋值给了变量 `a`（第 8 行）。这是把函数赋值给变量的语法。你如果观察得仔细的话，会发现赋值给 `a` 的函数没有名称。**由于没有名称，这类函数称为匿名函数（Anonymous Function）**。

调用该函数的唯一方法就是使用变量 `a`。我们在下一行调用了它。`a()` 调用了这个函数，打印出 `hello world first class function`。在第 12 行，我们打印出 `a` 的类型。这会输出 `func()`。

运行该程序，会输出：

```
Copyhello world first class function
func()
```

要调用一个匿名函数，可以不用赋值给变量。通过下面的例子，我们看看这是怎么做到的。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    func() {
        fmt.Println("hello world first class function")
    }()
}
```

在上面的程序中，第 8 行定义了一个匿名函数，并在定义之后，我们使用 `()` 立即调用了该函数（第 10 行）。该程序会输出：

```
Copyhello world first class function
```

就像其它函数一样，还可以向匿名函数传递参数。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    func(n string) {
        fmt.Println("Welcome", n)
    }("Gophers")
}
```

在上面的程序中，我们向匿名函数传递了一个字符串参数（第 10 行）。运行该程序后会输出：

```
CopyWelcome Gophers
```

## 用户自定义的函数类型

正如我们定义自己的结构体类型一样，我们可以定义自己的函数类型。

```go
Copytype add func(a int, b int) int
```

以上代码片段创建了一个新的函数类型 `add`，它接收两个整型参数，并返回一个整型。现在我们来定义 `add` 类型的变量。

我们来编写一个程序，定义一个 `add` 类型的变量。

```go
Copypackage main

import (  
    "fmt"
)

type add func(a int, b int) int

func main() {  
    var a add = func(a int, b int) int {
        return a + b
    }
    s := a(5, 6)
    fmt.Println("Sum", s)
}
```

在上面程序的第 10 行，我们定义了一个 `add` 类型的变量 `a`，并向它赋值了一个符合 `add` 类型签名的函数。我们在第 13 行调用了该函数，并将结果赋值给 `s`。该程序会输出：

```
CopySum 11
```

## 高阶函数

高阶函数（Hiher-order Function）定义为：**满足下列条件之一的函数**：

- **接收一个或多个函数作为参数**
- **返回值是一个函数**

针对上述两种情况，我们看看一些简单实例。

### 把函数作为参数，传递给其它函数

```go
Copypackage main

import (  
    "fmt"
)

func simple(a func(a, b int) int) {  
    fmt.Println(a(60, 7))
}

func main() {  
    f := func(a, b int) int {
        return a + b
    }
    simple(f)
}
```

在上面的实例中，第 7 行我们定义了一个函数 `simple`，`simple` 接收一个函数参数（该函数接收两个 `int` 参数，返回一个 `a` 整型）。在 `main` 函数的第 12 行，我们创建了一个匿名函数 `f`，其签名符合 `simple` 函数的参数。我们在下一行调用了 `simple`，并传递了参数 `f`。该程序打印输出 67。

### 在其它函数中返回函数

现在我们重写上面的代码，在 `simple` 函数中返回一个函数。

```go
Copypackage main

import (  
    "fmt"
)

func simple() func(a, b int) int {  
    f := func(a, b int) int {
        return a + b
    }
    return f
}

func main() {  
    s := simple()
    fmt.Println(s(60, 7))
}
```

在上面程序中，第 7 行的 `simple` 函数返回了一个函数，并接受两个 `int` 参数，返回一个 `int`。

在第 15 行，我们调用了 `simple` 函数。我们把 `simple` 的返回值赋值给了 `s`。现在 `s` 包含了 `simple` 函数返回的函数。我们调用了 `s`，并向它传递了两个 int 参数（第 16 行）。该程序输出 67。

## 闭包

闭包（Closure）是匿名函数的一个特例。当一个匿名函数所访问的变量定义在函数体的外部时，就称这样的匿名函数为闭包。

看看一个示例就明白了。

```go
Copypackage main

import (  
    "fmt"
)

func main() {  
    a := 5
    func() {
        fmt.Println("a =", a)
    }()
}
```

在上面的程序中，匿名函数在第 10 行访问了变量 `a`，而 `a` 存在于函数体的外部。因此这个匿名函数就是闭包。

每一个闭包都会绑定一个它自己的外围变量（Surrounding Variable）。我们通过一个简单示例来体会这句话的含义。

```go
Copypackage main

import (  
    "fmt"
)

func appendStr() func(string) string {  
    t := "Hello"
    c := func(b string) string {
        t = t + " " + b
        return t
    }
    return c
}

func main() {  
    a := appendStr()
    b := appendStr()
    fmt.Println(a("World"))
    fmt.Println(b("Everyone"))

    fmt.Println(a("Gopher"))
    fmt.Println(b("!"))
}
```

在上面程序中，函数 `appendStr` 返回了一个闭包。这个闭包绑定了变量 `t`。我们来理解这是什么意思。

在第 17 行和第 18 行声明的变量 `a` 和 `b` 都是闭包，它们绑定了各自的 `t` 值。

我们首先用参数 `World` 调用了 `a`。现在 `a` 中 `t` 值变为了 `Hello World`。

在第 20 行，我们又用参数 `Everyone` 调用了 `b`。由于 `b` 绑定了自己的变量 `t`，因此 `b` 中的 `t` 还是等于初始值 `Hello`。于是该函数调用之后，`b` 中的 `t` 变为了 `Hello Everyone`。程序的其他部分很简单，不再解释。

该程序会输出：

```
CopyHello World  
Hello Everyone  
Hello World Gopher  
Hello Everyone !
```

## 头等函数的实际用途

迄今为止，我们已经定义了什么是头等函数，也看了一些专门设计的示例，来学习它们如何工作。现在我们来编写一些实际的程序，来展现头等函数的实际用处。

我们会创建一个程序，基于一些条件，来过滤一个 `students` 切片。现在我们来逐步实现它。

首先定义一个 `student` 类型。

```go
Copytype student struct {  
    firstName string
    lastName string
    grade string
    country string
}
```

下一步是编写一个 `filter` 函数。该函数接收一个 `students` 切片和一个函数作为参数，这个函数会计算一个学生是否满足筛选条件。写出这个函数后，你很快就会明白，我们继续吧。

```go
Copyfunc filter(s []student, f func(student) bool) []student {  
    var r []student
    for _, v := range s {
        if f(v) == true {
            r = append(r, v)
        }
    }
    return r
}
```

在上面的函数中，`filter` 的第二个参数是一个函数。这个函数接收 `student` 参数，返回一个 `bool` 值。这个函数计算了某一学生是否满足筛选条件。我们在第 3 行遍历了 `student` 切片，将每个学生作为参数传递给了函数 `f`。如果该函数返回 `true`，就表示该学生通过了筛选条件，接着将该学生添加到了结果切片 `r` 中。你可能会很困惑这个函数的实际用途，等我们完成程序你就知道了。我添加了 `main` 函数，整个程序如下所示：

```go
Copypackage main

import (  
    "fmt"
)

type student struct {  
    firstName string
    lastName  string
    grade     string
    country   string
}

func filter(s []student, f func(student) bool) []student {  
    var r []student
    for _, v := range s {
        if f(v) == true {
            r = append(r, v)
        }
    }
    return r
}

func main() {  
    s1 := student{
        firstName: "Naveen",
        lastName:  "Ramanathan",
        grade:     "A",
        country:   "India",
    }
    s2 := student{
        firstName: "Samuel",
        lastName:  "Johnson",
        grade:     "B",
        country:   "USA",
    }
    s := []student{s1, s2}
    f := filter(s, func(s student) bool {
        if s.grade == "B" {
            return true
        }
        return false
    })
    fmt.Println(f)
}
```

在 `main` 函数中，我们首先创建了两个学生 `s1` 和 `s2`，并将他们添加到了切片 `s`。现在假设我们想要查询所有成绩为 `B` 的学生。为了实现这样的功能，我们传递了一个检查学生成绩是否为 `B` 的函数，如果是，该函数会返回 `true`。我们把这个函数作为参数传递给了 `filter` 函数（第 38 行）。上述程序会输出：

```
Copy[{Samuel Johnson B USA}]
```

假设我们想要查找所有来自印度的学生。通过修改传递给 `filter` 的函数参数，就很容易地实现了。

实现它的代码如下所示：

```go
Copyc := filter(s, func(s student) bool {  
    if s.country == "India" {
        return true
    }
    return false
})
fmt.Println(c)
```

请将该函数添加到 `main` 函数，并检查它的输出。

我们最后再编写一个程序，来结束这一节的讨论。这个程序会对切片的每个元素执行相同的操作，并返回结果。例如，如果我们希望将切片中的所有整数乘以 5，并返回出结果，那么通过头等函数可以很轻松地实现。我们把这种对集合中的每个元素进行操作的函数称为 `map` 函数。相关代码如下所示，它们很容易看懂。

```go
Copypackage main

import (  
    "fmt"
)

func iMap(s []int, f func(int) int) []int {  
    var r []int
    for _, v := range s {
        r = append(r, f(v))
    }
    return r
}
func main() {  
    a := []int{5, 6, 7, 8, 9}
    r := iMap(a, func(n int) int {
        return n * 5
    })
    fmt.Println(r)
}
```

该程序会输出：

```
Copy[25 30 35 40 45]
```

# 34. 反射

反射是 Go 语言的高级主题之一。

分为如下小节。

- 什么是反射？
- 为何需要检查变量，确定变量的类型？
- reflect 包
  - reflect.Type 和 reflect.Value
  - reflect.Kind
  - NumField() 和 Field() 方法
  - Int() 和 String() 方法
- 完整的程序
- 我们应该使用反射吗？

## 什么是反射？

反射就是程序能够在运行时检查变量和值，求出它们的类型。你可能还不太懂，这没关系。在本教程结束后，你就会清楚地理解反射，所以跟着我们的教程学习吧。

## 为何需要检查变量，确定变量的类型？

在学习反射时，所有人首先面临的疑惑就是：如果程序中每个变量都是我们自己定义的，那么在编译时就可以知道变量类型了，为什么我们还需要在运行时检查变量，求出它的类型呢？没错，在大多数时候都是这样，但并非总是如此。

我来解释一下吧。下面我们编写一个简单的程序。

```go
Copypackage main

import (
    "fmt"
)

func main() {
    i := 10
    fmt.Printf("%d %T", i, i)
}
```

在上面的程序中，`i` 的类型在编译时就知道了，然后我们在下一行打印出 `i`。这里没什么特别之处。

现在了解一下，需要在运行时求得变量类型的情况。假如我们要编写一个简单的函数，它接收结构体作为参数，并用它来创建一个 SQL 插入查询。

考虑下面的程序：

```go
Copypackage main

import (
    "fmt"
)

type order struct {
    ordId      int
    customerId int
}

func main() {
    o := order{
        ordId:      1234,
        customerId: 567,
    }
    fmt.Println(o)
}
```

在上面的程序中，我们需要编写一个函数，接收结构体变量 `o` 作为参数，返回下面的 SQL 插入查询。

```
Copyinsert into order values(1234, 567)
```

这个函数写起来很简单。我们现在编写这个函数。

```go
Copypackage main

import (
    "fmt"
)

type order struct {
    ordId      int
    customerId int
}

func createQuery(o order) string {
    i := fmt.Sprintf("insert into order values(%d, %d)", o.ordId, o.customerId)
    return i
}

func main() {
    o := order{
        ordId:      1234,
        customerId: 567,
    }
    fmt.Println(createQuery(o))
}
```

在第 12 行，`createQuery` 函数用 `o` 的两个字段（`ordId` 和 `customerId`），创建了插入查询。该程序会输出：

```bash
Copyinsert into order values(1234, 567)
```

现在我们来升级这个查询生成器。如果我们想让它变得通用，可以适用于任何结构体类型，该怎么办呢？我们用程序来理解一下。

```go
Copypackage main

type order struct {
    ordId      int
    customerId int
}

type employee struct {
    name string
    id int
    address string
    salary int
    country string
}

func createQuery(q interface{}) string {
}

func main() {

}
```

我们的目标就是完成 `createQuery` 函数（上述程序中的第 16 行），它可以接收任何结构体作为参数，根据结构体的字段创建插入查询。

例如，如果我们传入下面的结构体：

```go
Copyo := order {
    ordId: 1234,
    customerId: 567
}
```

`createQuery` 函数应该返回：

```
Copyinsert into order values (1234, 567)
```

类似地，如果我们传入：

```go
Copy e := employee {
        name: "Naveen",
        id: 565,
        address: "Science Park Road, Singapore",
        salary: 90000,
        country: "Singapore",
    }
```

该函数会返回：

```
Copyinsert into employee values("Naveen", 565, "Science Park Road, Singapore", 90000, "Singapore")
```

由于 `createQuery` 函数应该适用于任何结构体，因此它接收 `interface{}` 作为参数。为了简单起见，我们只处理包含 `string` 和 `int` 类型字段的结构体，但可以扩展为包含任何类型的字段。

`createQuery` 函数应该适用于所有的结构体。因此，要编写这个函数，就必须在运行时检查传递过来的结构体参数的类型，找到结构体字段，接着创建查询。这时就需要用到反射了。在本教程的下一步，我们将会学习如何使用 `reflect` 包来实现它。

## reflect 包

在 Go 语言中，`reflect`实现了运行时反射。`reflect` 包会帮助识别 `interface{}`变量的底层具体类型和具体值。这正是我们所需要的。`createQuery` 函数接收 `interface{}` 参数，根据它的具体类型和具体值，创建 SQL 查询。这正是 `reflect` 包能够帮助我们的地方。

在编写我们通用的查询生成器之前，我们首先需要了解 `reflect` 包中的几种类型和方法。让我们来逐个了解。

### reflect.Type 和 reflect.Value

`reflect.Type` 表示 `interface{}` 的具体类型，而 `reflect.Value` 表示它的具体值。`reflect.TypeOf()` 和 `reflect.ValueOf()` 两个函数可以分别返回 `reflect.Type` 和 `reflect.Value`。这两种类型是我们创建查询生成器的基础。我们现在用一个简单的例子来理解这两种类型。

```go
Copypackage main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

func createQuery(q interface{}) {
    t := reflect.TypeOf(q)
    v := reflect.ValueOf(q)
    fmt.Println("Type ", t)
    fmt.Println("Value ", v)


}
func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

}
```

在上面的程序中，第 13 行的 `createQuery` 函数接收 `interface{}` 作为参数。在第 14 行，`reflect.TypeOf` 接收了参数 `interface{}`，返回了`reflect.Type`，它包含了传入的 `interface{}` 参数的具体类型。同样地，在第 15 行，`reflect.ValueOf`函数接收参数 `interface{}`，并返回了 `reflect.Value`，它包含了传来的 `interface{}` 的具体值。

上述程序会打印：

```
CopyType  main.order
Value  {456 56}
```

从输出我们可以看到，程序打印了接口的具体类型和具体值。

### relfect.Kind

`reflect` 包中还有一个重要的类型：`Kind`。

在反射包中，`Kind` 和 `Type` 的类型可能看起来很相似，但在下面程序中，可以很清楚地看出它们的不同之处。

```go
Copypackage main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

func createQuery(q interface{}) {
    t := reflect.TypeOf(q)
    k := t.Kind()
    fmt.Println("Type ", t)
    fmt.Println("Kind ", k)


}
func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

}
```

上述程序会输出：

```
CopyType  main.order
Kind  struct
```

我想你应该很清楚两者的区别了。`Type` 表示 `interface{}` 的实际类型（在这里是 **main.Order**)，而 `Kind` 表示该类型的特定类别（在这里是 **struct**）。

### NumField() 和 Field() 方法

`NumField()`方法返回结构体中字段的数量，而 `Field(i int)`方法返回字段 `i` 的 `reflect.Value`。

```go
Copypackage main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

func createQuery(q interface{}) {
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        v := reflect.ValueOf(q)
        fmt.Println("Number of fields", v.NumField())
        for i := 0; i < v.NumField(); i++ {
            fmt.Printf("Field:%d type:%T value:%v\n", i, v.Field(i), v.Field(i))
        }
    }

}
func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)
}
```

在上面的程序中，因为 `NumField` 方法只能在结构体上使用，我们在第 14 行首先检查了 `q` 的类别是 `struct`。程序的其他代码很容易看懂，不作解释。该程序会输出：

```
CopyNumber of fields 2
Field:0 type:reflect.Value value:456
Field:1 type:reflect.Value value:56
```

### Int() 和 String() 方法

`Int`和 `String`可以帮助我们分别取出 `reflect.Value` 作为 `int64` 和 `string`。

```go
Copypackage main

import (
    "fmt"
    "reflect"
)

func main() {
    a := 56
    x := reflect.ValueOf(a).Int()
    fmt.Printf("type:%T value:%v\n", x, x)
    b := "Naveen"
    y := reflect.ValueOf(b).String()
    fmt.Printf("type:%T value:%v\n", y, y)

}
```

在上面程序中的第 10 行，我们取出 `reflect.Value`，并转换为 `int64`，而在第 13 行，我们取出 `reflect.Value` 并将其转换为 `string`。该程序会输出：

```
Copytype:int64 value:56
type:string value:Naveen
```

## 完整的程序

现在我们已经具备足够多的知识，来完成我们的查询生成器了，我们来实现它把。

```go
Copypackage main

import (
    "fmt"
    "reflect"
)

type order struct {
    ordId      int
    customerId int
}

type employee struct {
    name    string
    id      int
    address string
    salary  int
    country string
}

func createQuery(q interface{}) {
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        t := reflect.TypeOf(q).Name()
        query := fmt.Sprintf("insert into %s values(", t)
        v := reflect.ValueOf(q)
        for i := 0; i < v.NumField(); i++ {
            switch v.Field(i).Kind() {
            case reflect.Int:
                if i == 0 {
                    query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
                } else {
                    query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
                }
            case reflect.String:
                if i == 0 {
                    query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
                } else {
                    query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
                }
            default:
                fmt.Println("Unsupported type")
                return
            }
        }
        query = fmt.Sprintf("%s)", query)
        fmt.Println(query)
        return

    }
    fmt.Println("unsupported type")
}

func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

    e := employee{
        name:    "Naveen",
        id:      565,
        address: "Coimbatore",
        salary:  90000,
        country: "India",
    }
    createQuery(e)
    i := 90
    createQuery(i)

}
```

在第 22 行，我们首先检查了传来的参数是否是一个结构体。在第 23 行，我们使用了 `Name()` 方法，从该结构体的 `reflect.Type` 获取了结构体的名字。接下来一行，我们用 `t` 来创建查询。

在第 28 行，`case 语句`检查了当前字段是否为 `reflect.Int`，如果是的话，我们会取到该字段的值，并使用 `Int()` 方法转换为 `int64`。if else 语句用于处理边界情况。请添加日志来理解为什么需要它。在第 34 行，我们用来相同的逻辑来取到 `string`。

我们还作了额外的检查，以防止 `createQuery` 函数传入不支持的类型时，程序发生崩溃。程序的其他代码是自解释性的。我建议你在合适的地方添加日志，检查输出，来更好地理解这个程序。

该程序会输出：

```
Copyinsert into order values(456, 56)
insert into employee values("Naveen", 565, "Coimbatore", 90000, "India")
unsupported type
```

至于向输出的查询中添加字段名，我们把它留给读者作为练习。请尝试着修改程序，打印出以下格式的查询。

```
Copyinsert into order(ordId, customerId) values(456, 56)
```

## 我们应该使用反射吗？

我们已经展示了反射的实际应用，现在考虑一个很现实的问题。我们应该使用反射吗？我想引用 `Rob Pike`关于使用反射的格言，来回答这个问题。

> 清晰优于聪明。而反射并不是一目了然的。

反射是 Go 语言中非常强大和高级的概念，我们应该小心谨慎地使用它。使用反射编写清晰和可维护的代码是十分困难的。你应该尽可能避免使用它，只在必须用到它时，才使用反射。

# 35. 读取文件

文件读取是所有编程语言中最常见的操作之一。本教程我们会学习如何使用 Go 读取文件。

本教程分为如下小节。

- 将整个文件读取到内存
  - 使用绝对文件路径
  - 使用命令行标记来传递文件路径
  - 将文件绑定在二进制文件中
- 分块读取文件
- 逐行读取文件

## 将整个文件读取到内存

将整个文件读取到内存是最基本的文件操作之一。这需要使用 `ioutil`]包中的 [`ReadFile`](https://golang.org/pkg/io/ioutil/#ReadFile) 函数。

让我们在 Go 程序所在的目录中，读取一个文件。我已经在 GOPATH（译注：原文是 GOROOT，应该是笔误）中创建了文件夹，在该文件夹内部，有一个文本文件 `test.txt`，我们会使用 Go 程序 `filehandling.go` 来读取它。`test.txt` 包含文本 “Hello World. Welcome to file handling in Go”。我的文件夹结构如下：

```
Copysrc
    filehandling
        filehandling.go
        test.txt
```

接下来我们来看看代码。

```go
Copypackage main

import (
    "fmt"
    "io/ioutil"
)

func main() {
    data, err := ioutil.ReadFile("test.txt")
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

由于无法在 playground 上读取文件，因此请在你的本地环境运行这个程序。

在上述程序的第 9 行，程序会读取文件，并返回一个字节[切片](https://studygolang.com/articles/12121)，而这个切片保存在 `data` 中。在第 14 行，我们将 `data` 转换为 `string`，显示出文件的内容。

请在 **test.txt** 所在的位置运行该程序。

例如，对于 **linux/mac**，如果 **test.txt** 位于 **/home/naveen/go/src/filehandling**，可以使用下列步骤来运行程序。

```bash
Copy$ cd /home/naveen/go/src/filehandling/
$ go install filehandling
$ workspacepath/bin/filehandling
```

对于 **windows**，如果 **test.txt** 位于 **C:\Users\naveen.r\go\src\filehandling**，则使用下列步骤。

```bash
Copy> cd C:\Users\naveen.r\go\src\filehandling
> go install filehandling
> workspacepath\bin\filehandling.exe
```

该程序会输出：

```bas
CopyContents of file: Hello World. Welcome to file handling in Go.
```

如果在其他位置运行这个程序（比如 `/home/userdirectory`），会打印下面的错误。

```bash
CopyFile reading error open test.txt: The system cannot find the file specified.
```

这是因为 Go 是编译型语言。`go install` 会根据源代码创建一个二进制文件。二进制文件独立于源代码，可以在任何位置上运行。由于在运行二进制文件的位置上没有找到 `test.txt`，因此程序会报错，提示无法找到指定的文件。

有三种方法可以解决这个问题。

1. 使用绝对文件路径
2. 使用命令行标记来传递文件路径
3. 将文件绑定在二进制文件中

让我们来依次介绍。

### 1. 使用绝对文件路径

要解决问题，最简单的方法就是传入绝对文件路径。我已经修改了程序，把路径改成了绝对路径。

```go
Copypackage main

import (
    "fmt"
    "io/ioutil"
)

func main() {
    data, err := ioutil.ReadFile("/home/naveen/go/src/filehandling/test.txt")
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

现在可以在任何位置上运行程序，打印出 `test.txt` 的内容。

例如，可以在我的家目录运行。

```bash
Copy$ cd $HOME
$ go install filehandling
$ workspacepath/bin/filehandling
```

该程序打印出了 `test.txt` 的内容。

看似这是一个简单的方法，但它的缺点是：文件必须放在程序指定的路径中，否则就会出错。

### 2. 使用命令行标记来传递文件路径

另一种解决方案是使用命令行标记来传递文件路径。使用 [flag](https://golang.org/pkg/flag/) 包，我们可以从输入的命令行获取到文件路径，接着读取文件内容。

首先我们来看看 `flag` 包是如何工作的。`flag` 包有一个名为 [`String`](https://golang.org/pkg/flag/#String) 的[函数](https://studygolang.com/articles/11892)。该函数接收三个参数。第一个参数是标记名，第二个是默认值，第三个是标记的简短描述。

让我们来编写程序，从命令行读取文件名。将 `filehandling.go` 的内容替换如下：

```go
Copypackage main
import (
    "flag"
    "fmt"
)

func main() {
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()
    fmt.Println("value of fpath is", *fptr)
}
```

在上述程序中第 8 行，通过 `String` 函数，创建了一个字符串标记，名称是 `fpath`，默认值是 `test.txt`，描述为 `file path to read from`。这个函数返回存储 flag 值的字符串[变量](https://studygolang.com/articles/11756)的地址。

在程序访问 flag 之前，必须先调用 `flag.Parse()`。

在第 10 行，程序会打印出 flag 值。

使用下面命令运行程序。

```bash
Copywrkspacepath/bin/filehandling -fpath=/path-of-file/test.txt
```

我们传入 `/path-of-file/test.txt`，赋值给了 `fpath` 标记。

该程序输出：

```bash
Copyvalue of fpath is /path-of-file/test.txt
```

这是因为 `fpath` 的默认值是 `test.txt`。

现在我们知道如何从命令行读取文件路径了，让我们继续完成我们的文件读取程序。

```go
Copypackage main
import (
    "flag"
    "fmt"
    "io/ioutil"
)

func main() {
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()
    data, err := ioutil.ReadFile(*fptr)
    if err != nil {
        fmt.Println("File reading error", err)
        return
    }
    fmt.Println("Contents of file:", string(data))
}
```

在上述程序里，命令行传入文件路径，程序读取了该文件的内容。使用下面命令运行该程序。

```bash
Copywrkspacepath/bin/filehandling -fpath=/path-of-file/test.txt
```

请将 `/path-of-file/` 替换为 `test.txt` 的真实路径。该程序将打印：

```txt
CopyContents of file: Hello World. Welcome to file handling in Go.
```

### 3. 将文件绑定在二进制文件中

虽然从命令行获取文件路径的方法很好，但还有一种更好的解决方法。如果我们能够将文本文件捆绑在二进制文件，岂不是很棒？这就是我们下面要做的事情。

有很多[包](https://studygolang.com/articles/11893)可以帮助我们实现。我们会使用 [packr](https://github.com/gobuffalo/packr)，因为它很简单，并且我在项目中使用它时，没有出现任何问题。

第一步就是安装 `packr` 包。

在命令提示符中输入下面命令，安装 `packr` 包。

```bash
Copygo get -u github.com/gobuffalo/packr/...
```

`packr` 会把静态文件（例如 `.txt` 文件）转换为 `.go` 文件，接下来，`.go` 文件会直接嵌入到二进制文件中。`packer` 非常智能，在开发过程中，可以从磁盘而非二进制文件中获取静态文件。在开发过程中，当仅仅静态文件变化时，可以不必重新编译。

我们通过程序来更好地理解它。用以下内容来替换 `handling.go` 文件。

```go
Copypackage main

import (
    "fmt"

    "github.com/gobuffalo/packr"
)

func main() {
    box := packr.NewBox("../filehandling")
    data := box.String("test.txt")
    fmt.Println("Contents of file:", data)
}
```

在上面程序的第 10 行，我们创建了一个新盒子（New Box）。盒子表示一个文件夹，其内容会嵌入到二进制中。在这里，我指定了 `filehandling` 文件夹，其内容包含 `test.txt`。在下一行，我们读取了文件内容，并打印出来。

在开发阶段时，我们可以使用 `go install` 命令来运行程序。程序可以正常运行。`packr` 非常智能，在开发阶段可以从磁盘加载文件。

使用下面命令来运行程序。

```bash
Copygo install filehandling
workspacepath/bin/filehandling
```

该命令可以在其他位置运行。`packr` 很聪明，可以获取传递给 `NewBox` 命令的目录的绝对路径。

该程序会输出：

```txt
CopyContents of file: Hello World. Welcome to file handling in Go.
```

你可以试着改变 `test.txt` 的内容，然后再运行 `filehandling`。可以看到，无需再次编译，程序打印出了 `test.txt` 的更新内容。完美！

现在我们来看看如何将 `test.txt` 打包到我们的二进制文件中。我们使用 `packr` 命令来实现。

运行下面的命令：

```bash
Copypackr install -v filehandling
```

它会打印：

```bash
Copybuilding box ../filehandling
packing file filehandling.go
packed file filehandling.go
packing file test.txt
packed file test.txt
built box ../filehandling with ["filehandling.go" "test.txt"]
filehandling
```

该命令将静态文件绑定到了二进制文件中。

在运行上述命令之后，使用命令 `workspacepath/bin/filehandling` 来运行程序。程序会打印出 `test.txt` 的内容。于是从二进制文件中，我们读取了 `test.txt` 的内容。

如果你不知道文件到底是由二进制还是磁盘来提供，我建议你删除 `test.txt`，并在此运行 `filehandling` 命令。你将看到，程序打印出了 `test.txt` 的内容。太棒了:D。我们已经成功将静态文件嵌入到了二进制文件中。

## 分块读取文件

在前面的章节，我们学习了如何把整个文件读取到内存。当文件非常大时，尤其在 RAM 存储量不足的情况下，把整个文件都读入内存是没有意义的。更好的方法是分块读取文件。这可以使用 [bufio](https://golang.org/pkg/bufio) 包来完成。

让我们来编写一个程序，以 3 个字节的块为单位读取 `test.txt` 文件。如下所示，替换 `filehandling.go` 的内容。 [Go-2.进阶.md](Go-2.进阶.md) 

```go
Copypackage main

import (
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
)

func main() {
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()

    f, err := os.Open(*fptr)
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        if err = f.Close(); err != nil {
            log.Fatal(err)
        }
    }()
    r := bufio.NewReader(f)
    b := make([]byte, 3)
    for {
        _, err := r.Read(b)
        if err != nil {
            fmt.Println("Error reading file:", err)
            break
        }
        fmt.Println(string(b))
    }
}
```

在上述程序的第 15 行，我们使用命令行标记传递的路径，打开文件。

在第 19 行，我们延迟了文件的关闭操作。

在上面程序的第 24 行，我们新建了一个缓冲读取器（buffered reader）。在下一行，我们创建了长度和容量为 3 的字节切片，程序会把文件的字节读取到切片中。

第 27 行的 `Read` [方法](https://studygolang.com/articles/12264)会读取 len(b) 个字节（达到 3 字节），并返回所读取的字节数。当到达文件最后时，它会返回一个 EOF 错误。程序的其他地方比较简单，不做解释。

如果我们使用下面命令来运行程序：

```bash
Copy$ go install filehandling
$ wrkspacepath/bin/filehandling -fpath=/path-of-file/test.txt
```

会得到以下输出：

```bash
CopyHel
lo
Wor
ld.
 We
lco
me
to
fil
e h
and
lin
g i
n G
o.
Error reading file: EOF
```

## 逐行读取文件

本节我们讨论如何使用 Go 逐行读取文件。这可以使用 [bufio](https://golang.org/pkg/bufio/) 来实现。

请将 `test.txt` 替换为以下内容。

```
CopyHello World. Welcome to file handling in Go.
This is the second line of the file.
We have reached the end of the file.
```

逐行读取文件涉及到以下步骤。

1. 打开文件；
2. 在文件上新建一个 scanner；
3. 扫描文件并且逐行读取。

将 `filehandling.go` 替换为以下内容。

```go
Copypackage main

import (
    "bufio"
    "flag"
    "fmt"
    "log"
    "os"
)

func main() {
    fptr := flag.String("fpath", "test.txt", "file path to read from")
    flag.Parse()

    f, err := os.Open(*fptr)
    if err != nil {
        log.Fatal(err)
    }
    defer func() {
        if err = f.Close(); err != nil {
        log.Fatal(err)
    }
    }()
    s := bufio.NewScanner(f)
    for s.Scan() {
        fmt.Println(s.Text())
    }
    err = s.Err()
    if err != nil {
        log.Fatal(err)
    }
}
```

在上述程序的第 15 行，我们用命令行标记传入的路径，打开文件。在第 24 行，我们用文件创建了一个新的 scanner。第 25 行的 `Scan()` 方法读取文件的下一行，如果可以读取，就可以使用 `Text()` 方法。

当 `Scan` 返回 false 时，除非已经到达文件末尾（此时 `Err()` 返回 `nil`），否则 `Err()` 就会返回扫描过程中出现的错误。

如果我使用下面命令来运行程序：

```bash
Copy$ go install filehandling
$ workspacepath/bin/filehandling -fpath=/path-of-file/test.txt
```

程序会输出：

```bash
CopyHello World. Welcome to file handling in Go.
This is the second line of the file.
We have reached the end of the file.
```

## 将字符串写入文件

最常见的写文件就是将字符串写入文件。这个写起来非常的简单。这个包含以下几个阶段。

1. 创建文件
2. 将字符串写入文件

我们将得到如下代码。

```go
Copypackage main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Create("test.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    l, err := f.WriteString("Hello World")
    if err != nil {
        fmt.Println(err)
        f.Close()
        return
    }
    fmt.Println(l, "bytes written successfully")
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

在第 9 行使用 `create` 创建一个名字为 `test.txt` 的文件。如果这个文件已经存在，那么 `create` 函数将截断这个文件。该函数返回一个文件描述符。

在第 14 行，我们使用 `WriteString` 将字符串 **Hello World** 写入到文件里面。这个方法将返回相应写入的字节数，如果有错误则返回错误。

最后，在第 21 行我们将文件关闭。

上面程序将打印：

```
Copy11 bytes written successfully
```

运行完成之后你会在程序运行的目录下发现创建了一个 **test.txt** 的文件。如果你使用文本编辑器打开这个文件，你可以看到文件里面有一个 **Hello World** 的字符串。

## 将字节写入文件

将字节写入文件和写入字符串非常的类似。我们将使用 Write]方法将字节写入到文件。下面的程序将一个字节的切片写入文件。

```go
Copypackage main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Create("/home/naveen/bytes")
    if err != nil {
        fmt.Println(err)
        return
    }
    d2 := []byte{104, 101, 108, 108, 111, 32, 119, 111, 114, 108, 100}
    n2, err := f.Write(d2)
    if err != nil {
        fmt.Println(err)
        f.Close()
        return
    }
    fmt.Println(n2, "bytes written successfully")
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

在上面的程序中，第 15 行使用了 **Write** 方法将字节切片写入到 `bytes` 这个文件里。这个文本在目录 `/home/naveen` 里面。你也可以将这个目录换成其他的目录。剩余的程序自带解释。如果执行成功，这个程序将打印 `11 bytes written successfully`。并且创建一个 `bytes` 的文件。打开文件，你会发现该文件包含了文本 **hello bytes**。

## 将字符串一行一行的写入文件

另外一个常用的操作就是将字符串一行一行的写入到文件。这一部分我们将写一个程序，该程序创建并写入如下内容到文件里。

```
CopyWelcome to the world of Go.
Go is a compiled language.
It is easy to learn Go.
```

让我们看下面的代码：

```go
Copypackage main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Create("lines")
    if err != nil {
        fmt.Println(err)
        f.Close()
        return
    }
    d := []string{"Welcome to the world of Go1.", "Go is a compiled language.",
"It is easy to learn Go."}

    for _, v := range d {
        fmt.Fprintln(f, v)
        if err != nil {
            fmt.Println(err)
            return
        }
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("file written successfully")
}
```

在上面程序的第 9 行，我们先创建一个名字叫做 **lines** 的文件。在第 17 行，我们用迭代并使用 `for rang` 循环这个数组，并使用 Fprintln，**Fprintln** 函数 将 `io.writer` 做为参数，并且添加一个新的行，这个正是我们想要的。如果执行成功将打印 `file written successfully`，并且在当前目录将创建一个 `lines` 的文件。`lines` 这个文件的内容如下所示：

```
CopyWelcome to the world of Go1.
Go is a compiled language.
It is easy to learn Go.
```

## 追加到文件

这一部分我们将追加一行到上节创建的 `lines` 文件中。我们将追加 **File handling is easy** 到 `lines` 这个文件。

这个文件将以追加和写的方式打开。这些标志将通过 Open方法实现。当文件以追加的方式打开，我们添加新的行到文件里。

```go
Copypackage main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.OpenFile("lines", os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        fmt.Println(err)
        return
    }
    newLine := "File handling is easy."
    _, err = fmt.Fprintln(f, newLine)
    if err != nil {
        fmt.Println(err)
                f.Close()
        return
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("file appended successfully")
}
```

在上面程序的第 9 行，我们以写的方式打开文件并将一行添加到文件里。当成功打开文件之后，在程序第 15 行，我们添加一行到文件里。程序成功将打印 `file appended successfully`。运行程序，新的行就加到文件里面去了。

```
CopyWelcome to the world of Go1.
Go is a compiled language.
It is easy to learn Go.
File handling is easy.
```

## 并发写文件

当多个 goroutines 同时（并发）写文件时，我们会遇到竞争条件(race condition)。因此，当发生同步写的时候需要一个 channel 作为一致写入的条件。

我们将写一个程序，该程序创建 100 个 goroutinues。每个 goroutinue 将并发产生一个随机数，届时将有 100 个随机数产生。这些随机数将被写入到文件里面。我们将用下面的方法解决这个问题 .

1. 创建一个 channel 用来读和写这个随机数。
2. 创建 100 个生产者 goroutine。每个 goroutine 将产生随机数并将随机数写入到 channel 里。
3. 创建一个消费者 goroutine 用来从 channel 读取随机数并将它写入文件。这样的话我们就只有一个 goroutinue 向文件中写数据，从而避免竞争条件。
4. 一旦完成则关闭文件。

我们开始写产生随机数的 `produce` 函数：

```go
Copyfunc produce(data chan int, wg *sync.WaitGroup) {
    n := rand.Intn(999)
    data <- n
    wg.Done()
}
```

上面的方法产生随机数并且将 `data` 写入到 channel 中，之后通过调用 `waitGroup` 的 `Done` 方法来通知任务已经完成。

让我们看看将数据写到文件的函数：

```go
Copyfunc consume(data chan int, done chan bool) {
    f, err := os.Create("concurrent")
    if err != nil {
        fmt.Println(err)
        return
    }
    for d := range data {
        _, err = fmt.Fprintln(f, d)
        if err != nil {
            fmt.Println(err)
            f.Close()
            done <- false
            return
        }
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        done <- false
        return
    }
    done <- true
}
```

这个 `consume` 的函数创建了一个名为 `concurrent` 的文件。然后从 channel 中读取随机数并且写到文件中。一旦读取完成并且将随机数写入文件后，通过往 `done` 这个 cahnnel 中写入 `true` 来通知任务已完成。

下面我们写 `main` 函数，并完成这个程序。下面是我提供的完整程序：

```go
Copypackage main

import (
    "fmt"
    "math/rand"
    "os"
    "sync"
)

func produce(data chan int, wg *sync.WaitGroup) {
    n := rand.Intn(999)
    data <- n
    wg.Done()
}

func consume(data chan int, done chan bool) {
    f, err := os.Create("concurrent")
    if err != nil {
        fmt.Println(err)
        return
    }
    for d := range data {
        _, err = fmt.Fprintln(f, d)
        if err != nil {
            fmt.Println(err)
            f.Close()
            done <- false
            return
        }
    }
    err = f.Close()
    if err != nil {
        fmt.Println(err)
        done <- false
        return
    }
    done <- true
}

func main() {
    data := make(chan int)
    done := make(chan bool)
    wg := sync.WaitGroup{}
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go produce(data, &wg)
    }
    go consume(data, done)
    go func() {
        wg.Wait()
        close(data)
    }()
    d := <-done
    if d == true {
        fmt.Println("File written successfully")
    } else {
        fmt.Println("File writing failed")
    }
}
```

`main` 函数在第 41 行创建写入和读取数据的 channel，在第 42 行创建 `done` 这个 channel，此 channel 用于消费者 goroutinue 完成任务之后通知 `main` 函数。第 43 行创建 Waitgroup 的实例 `wg`，用于等待所有生产随机数的 goroutine 完成任务。

在第 44 行使用 `for` 循环创建 100 个 goroutines。在第 49 行调用 waitgroup 的 `wait()` 方法等待所有的 goroutines 完成随机数的生成。然后关闭 channel。当 channel 关闭时，消费者 `consume` goroutine 已经将所有的随机数写入文件，在第 37 行 将 `true` 写入 `done` 这个 channel 中，这个时候 `main` 函数解除阻塞并且打印 `File written successfully`。
