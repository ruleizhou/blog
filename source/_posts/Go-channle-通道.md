---
title: Go channle 通道
date: 2019-12-26 11:04:30
categories:
- Go
tags:
- Go
typora-root-url: Go-channle-通道
typora-copy-images-to: Go-channle-通道
---

通道可以被认为是Goroutines通信的管道。类似于管道中的水从一端到另一端的流动，数据可以从一端发送到另一端，通过通道接收。

在前面讲Go语言的并发时候，我们就说过，当多个Goroutine想实现共享数据的时候，虽然也提供了传统的同步机制，但是Go语言强烈建议的是使用Channel通道来实现Goroutines之间的通信。

> “不要通过共享内存来通信，而应该通过通信来共享内存” 这是一句风靡golang社区的经典语

Go语言中，要传递某个数据给另一个goroutine(协程)，可以把这个数据封装成一个对象，然后把这个对象的指针传入某个channel中，另外一个goroutine从这个channel中读出这个指针，并处理其指向的内存对象。Go从语言层面保证同一个时间只有一个goroutine能够访问channel里面的数据，为开发者提供了一种优雅简单的工具，所以Go的做法就是使用channel来通信，通过通信来传递内存数据，使得内存数据在不同的goroutine中传递，而不是使用共享内存来通信。

# 概述

## 通道的概念

通道是什么，通道就是goroutine之间的通道。它可以让goroutine之间相互通信。

每个通道都有与其相关的类型。该类型是通道允许传输的数据类型。(通道的零值为nil。nil通道没有任何用处，因此通道必须使用类似于map和切片的方法来定义。)

## 通道的声明

声明一个通道和定义一个变量的语法一样：

```go
//声明通道
var 通道名 chan 数据类型
//创建通道：如果通道为nil(就是不存在)，就需要先创建通道
通道名 = make(chan 数据类型)
```

示例代码：

```go
package main

import "fmt"

func main() {
    var a chan int
    if a == nil {
        fmt.Println("channel 是 nil 的, 不能使用，需要先创建通道。。")
        a = make(chan int)
        fmt.Printf("数据类型是： %T", a)
    }
}
```

运行结果：

```shell
channel 是 nil 的, 不能使用，需要先创建通道。。
数据类型是： chan int
```

也可以简短的声明：

```go
a := make(chan int) 
```

## channel的数据类型

channel是引用类型的数据，在作为参数传递的时候，传递的是内存地址。

示例代码：

```go
package main

import (
    "fmt"
)

func main() {
    ch1 := make(chan int)
    fmt.Printf("%T,%p\n",ch1,ch1)

    test1(ch1)

}

func test1(ch chan int){
    fmt.Printf("%T,%p\n",ch,ch)
}
```

我们能够看到，ch和ch1的地址是一样的，说明它们是同一个通道。

## channel注意事项

Channel通道在使用的时候，有以下几个注意点：

- 1.用于goroutine，传递消息的。
- 2.通道，每个都有相关联的数据类型,
  nil chan，不能使用，类似于nil map，不能直接存储键值对
- 3.使用通道传递数据：<-
  chan <- data,发送数据到通道。向通道中写数据
  data <- chan,从通道中获取数据。从通道中读数据
- 4.阻塞：
  发送数据：chan <- data,阻塞的，直到另一条goroutine，读取数据来解除阻塞
  读取数据：data <- chan,也是阻塞的。直到另一条goroutine，写出数据解除阻塞。
- 5.本身channel就是同步的，意味着同一时间，只能有一条goroutine来操作。

最后：通道是goroutine之间的连接，所以通道的发送和接收必须处在不同的goroutine中。



# 通道的使用

## 发送和接收

语法：

```go
data := <- a // read from channel a  
a <- data // write to channel a
```

```go
v, ok := <- a //从一个channel中读取
```

一个通道发送和接收数据，默认是阻塞的。当一个数据被发送到通道时，在发送语句中被阻塞，直到另一个Goroutine从该通道读取数据。相对地，当从通道读取数据时，读取被阻塞，直到一个Goroutine将数据写入该通道。

这些通道的特性是帮助Goroutines有效地进行通信，而无需像使用其他编程语言中非常常见的显式锁或条件变量。

```go
package main

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

## 关闭通道

发送者可以通过关闭信道，来通知接收方不会有更多的数据被发送到channel上。

```go
close(ch)
```

接收者可以在接收来自通道的数据时使用额外的变量来检查通道是否已经关闭。

```go
v, ok := <- ch  
```

> 类似map操作，存储key，value键值对
>
> v,ok := map[key] //根据key从map中获取value，如果key存在， v就是对应的数据，如果key不存在，v是默认值

在上面的语句中，如果ok的值是true，表示成功的从通道中读取了一个数据value。如果ok是false，这意味着我们正在从一个封闭的通道读取数据。从闭通道读取的值将是通道类型的零值。

示例代码：

```go
package main

import (
    "fmt"
    "time"
)

func main()  {
    ch1 := make(chan int)
    go sendData(ch1)
    /*
    子goroutine，写出数据10个
            每写一个，阻塞一次，主程序读取一次，解除阻塞

    主goroutine：循环读
            每次读取一个，堵塞一次，子程序，写出一个，解除阻塞

    发送发，关闭通道的--->接收方，接收到的数据是该类型的零值，以及false
     */
    //主程序中获取通道的数据
    for{
        time.Sleep(1*time.Second)
        v, ok := <- ch1 //其他goroutine，显示的调用close方法关闭通道。
        if !ok{
            fmt.Println("已经读取了所有的数据，", ok)
            break
        }
        fmt.Println("取出数据：",v, ok)
    }

    fmt.Println("main...over....")
}
func sendData(ch1 chan int)  {
    // 发送方：10条数据
    for i:=0;i<10 ;i++  {
        ch1 <- i//将i写入通道中
    }
    close(ch1) //将ch1通道关闭了。
}
```

send Goroutine将0到9写入chl通道，然后关闭通道。主函数里有一个无限循环。它检查通道是否在发送数据后，使用变量ok关闭。如果ok是假的，则意味着通道关闭，因此循环结束。还可以打印接收到的值和ok的值。

## 通道上的范围循环

我们可以循环从通道上获取数据，直到通道关闭。for循环的for range形式可用于从通道接收值，直到它关闭为止。

示例代码：

```go
package main

import (
    "time"
    "fmt"
)

func main()  {
    ch1 :=make(chan int)
    go sendData(ch1)
    // for循环的for range形式可用于从通道接收值，直到它关闭为止。
    for v := range ch1{
        fmt.Println("读取数据：",v)
    }
    fmt.Println("main..over.....")
}
func sendData(ch1 chan int)  {
    for i:=0;i<10 ; i++ {
        time.Sleep(1*time.Second)
        ch1 <- i
    }
    close(ch1)//通知对方，通道关闭
}
```

运行结果：

```shell
读取数据： 0
读取数据： 1
读取数据： 2
读取数据： 3
读取数据： 4
读取数据： 5
读取数据： 6
读取数据： 7
读取数据： 8
读取数据： 9
main..over.....
```

## 死锁

使用通道时要考虑的一个重要因素是死锁。如果Goroutine在一个通道上发送数据，那么预计其他的Goroutine应该接收数据。如果这种情况不发生，那么程序将在运行时出现死锁。

类似地，如果Goroutine正在等待从通道接收数据，那么另一些Goroutine将会在该通道上写入数据，否则程序将会死锁。

示例代码：

```go
package main

func main() {  
    ch := make(chan int)
    ch <- 5
}
```

报错：

```shell
fatal error: all goroutines are asleep - deadlock!
```

> 在主流的编程语言中为了保证多线程之间共享数据安全性和一致性，都会提供一套基本的同步工具集，如锁，条件变量，原子操作等等。Go语言标准库也毫不意外的提供了这些同步机制，使用方式也和其他语言也差不多。
>
> 除了这些基本的同步手段，Go语言还提供了一种新的同步机制: Channel，它在Go语言中是一个像int, float32等的基本类型，一个channel可以认为是一个能够在多个Goroutine之间传递某一类型的数据的管道。Go中的channel无论是实现机制还是使用场景都和Java中的BlockingQueue很接近。



# 缓冲通道

缓冲通道就是指一个通道，带有一个缓冲区。发送到一个缓冲通道只有在缓冲区满时才被阻塞。类似地，从缓冲通道接收的信息只有在缓冲区为空时才会被阻塞。

可以通过将额外的容量参数传递给make函数来创建缓冲通道，该函数指定缓冲区的大小。

```go
ch := make(chan type, capacity)  
```

上述语法的容量应该大于0，以便通道具有缓冲区。默认情况下，无缓冲通道的容量为0，因此在之前创建通道时省略了容量参数。

示例代码：

```go
package main

import (
    "fmt"
    "strconv"
    "time"
)

func main() {
    /*
    非缓存通道：make(chan T)
    缓存通道：make(chan T ,size)
        缓存通道，理解为是队列：

    非缓存，发送还是接受，都是阻塞的
    缓存通道,缓存区的数据满了，才会阻塞状态。。

     */
    ch1 := make(chan int)           //非缓存的通道
    fmt.Println(len(ch1), cap(ch1)) //0 0
    //ch1 <- 100//阻塞的，需要其他的goroutine解除阻塞，否则deadlock

    ch2 := make(chan int, 5)        //缓存的通道，缓存区大小是5
    fmt.Println(len(ch2), cap(ch2)) //0 5
    ch2 <- 100                      //
    fmt.Println(len(ch2), cap(ch2)) //1 5

    //ch2 <- 200
    //ch2 <- 300
    //ch2 <- 400
    //ch2 <- 500
    //ch2 <- 600
    fmt.Println("--------------")
    ch3 := make(chan string, 4)
    go sendData3(ch3)
    for {
        time.Sleep(1*time.Second)
        v, ok := <-ch3
        if !ok {
            fmt.Println("读完了，，", ok)
            break
        }
        fmt.Println("\t读取的数据是：", v)
    }

    fmt.Println("main...over...")
}

func sendData3(ch3 chan string) {
    for i := 0; i < 10; i++ {
        ch3 <- "数据" + strconv.Itoa(i)
        fmt.Println("子goroutine，写出第", i, "个数据")
    }
    close(ch3)
}
```

运行结果：

```shell
0 0
0 5
1 5
--------------
子goroutine，写出第 0 个数据
子goroutine，写出第 1 个数据
子goroutine，写出第 2 个数据
子goroutine，写出第 3 个数据
子goroutine，写出第 4 个数据
	读取的数据是： 数据0
	读取的数据是： 数据1
子goroutine，写出第 5 个数据
	读取的数据是： 数据2
子goroutine，写出第 6 个数据
	读取的数据是： 数据3
子goroutine，写出第 7 个数据
	读取的数据是： 数据4
子goroutine，写出第 8 个数据
	读取的数据是： 数据5
子goroutine，写出第 9 个数据
	读取的数据是： 数据6
	读取的数据是： 数据7
	读取的数据是： 数据8
	读取的数据是： 数据9
读完了，， false
main...over...
```



# 定向通道

通道，channel，是用于实现goroutine之间的通信的。一个goroutine可以向通道中发送数据，另一条goroutine可以从该通道中获取数据。截止到现在我们所学习的通道，都是既可以发送数据，也可以读取数据，我们又把这种通道叫做双向通道。

```go
data := <- a // read from channel a  
a <- data // write to channel a
```

单向通道，也就是定向通道。

之前我们学习的通道都是双向通道，我们可以通过这些通道接收或者发送数据。我们也可以创建单向通道，这些通道只能发送或者接收数据。

```go
package main

import "fmt"

func main()  {
    /*
        单向：定向
        chan <- T,
            只支持写，
        <- chan T,
            只读

        用于参数传递：
     */
    ch1 := make(chan int)//双向，读，写
    //ch2 := make(chan <- int) // 单向，只写，不能读
    //ch3 := make(<- chan int) //单向，只读，不能写
    //ch1 <- 100
    //data :=<-ch1
    //ch2 <- 1000
    //data := <- ch2
    //fmt.Println(data)
    //  <-ch2 //invalid operation: <-ch2 (receive from send-only type chan<- int)
    //ch3 <- 100
    //  <-ch3
    //  ch3 <- 100 //invalid operation: ch3 <- 100 (send to receive-only type <-chan int)

    //go fun1(ch2)
    go fun1(ch1)
    data:= <- ch1
    fmt.Println("fun1中写出的数据是：",data)

    //fun2(ch3)
    go fun2(ch1)
    ch1 <- 200
    fmt.Println("main。。over。。")
}
//该函数接收，只写的通道
func fun1(ch chan <- int){
    // 函数内部，对于ch只能写数据，不能读数据
    ch <- 100
    fmt.Println("fun1函数结束。。")
}

func fun2(ch <-chan int){
    //函数内部，对于ch只能读数据，不能写数据
    data := <- ch
    fmt.Println("fun2函数，从ch中读取的数据是：",data)
}
```



# 参考

> 1. Golang中国，https://www.qfgolang.com/