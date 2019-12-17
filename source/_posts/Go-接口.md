---
title: Go 接口
date: 2019-12-17 10:55:54
categories:
- Go
tags:
- Go
typora-root-url: Go-接口
typora-copy-images-to: Go-接口
---

# 接口的使用

## 什么是接口

面向对象世界中的接口的一般定义是“接口定义对象的行为”。它表示让指定对象应该做什么。实现这种行为的方法(实现细节)是针对对象的。

在Go中，接口是一组方法签名。当类型为接口中的所有方法提供定义时，它被称为实现接口。它与OOP非常相似。接口指定了类型应该具有的方法，类型决定了如何实现这些方法。

> 它把所有的具有共性的方法定义在一起，任何其他类型只要实现了这些方法就是实现了这个接口
>
> 接口定义了一组方法，如果某个对象实现了某个接口的所有方法，则此对象就实现了该接口。

## 接口的定义

定义接口：

```go
/* 定义接口 */
type interface_name interface {
   method_name1 [return_type]
   method_name2 [return_type]
   method_name3 [return_type]
   ...
   method_namen [return_type]
}

/* 定义结构体 */
type struct_name struct {
   /* variables */
}

/* 实现接口方法 */
func (struct_name_variable struct_name) method_name1() [return_type] {
   /* 方法实现 */
}
...
func (struct_name_variable struct_name) method_namen() [return_type] {
   /* 方法实现*/
}
```

示例代码：

```go
package main

import (
    "fmt"
)

type Phone interface {
    call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
    fmt.Println("I am Nokia, I can call you!")
}

type IPhone struct {
}

func (iPhone IPhone) call() {
    fmt.Println("I am iPhone, I can call you!")
}

func main() {
    var phone Phone

    phone = new(NokiaPhone)
    phone.call()

    phone = new(IPhone)
    phone.call()

}
```

运行结果：

```shell
I am Nokia, I can call you!
I am iPhone, I can call you!
```

- interface可以被任意的对象实现
- 一个对象可以实现任意多个interface
- 任意的类型都实现了空interface(我们这样定义：interface{})，也就是包含0个method的interface

## interface值

```go
package main

import "fmt"

type Human struct {
    name  string
    age   int
    phone string
}
type Student struct {
    Human  //匿名字段
    school string
    loan   float32
}
type Employee struct {
    Human   //匿名字段
    company string
    money   float32
} //Human实现Sayhi方法
func (h Human) SayHi() {
    fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
} //Human实现Sing方法
func (h Human) Sing(lyrics string) {
    fmt.Println("La la la la...", lyrics)
} //Employee重写Human的SayHi方法
func (e Employee) SayHi() {
    fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
        e.company, e.phone) //Yes you can split into 2 lines here.
}

// Interface Men被Human,Student和Employee实现
// 因为这三个类型都实现了这两个方法
type Men interface {
    SayHi()
    Sing(lyrics string)
}

func main() {
    mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
    paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
    sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
    Tom := Employee{Human{"Sam", 36, "444-222-XXX"}, "Things Ltd.", 5000}
    //定义Men类型的变量i
    var i Men
    //i能存储Student
    i = mike
    fmt.Println("This is Mike, a Student:")
    i.SayHi()
    i.Sing("November rain")
    //i也能存储Employee
    i = Tom
    fmt.Println("This is Tom, an Employee:")
    i.SayHi()
    i.Sing("Born to be wild")
    //定义了slice Men
    fmt.Println("Let's use a slice of Men and see what happens")
    x := make([]Men, 3)
    //T这三个都是不同类型的元素，但是他们实现了interface同一个接口
    x[0], x[1], x[2] = paul, sam, mike
    for _, value := range x {
        value.SayHi()
    }
}
```

运行结果：

```go
    This is Mike, a Student:
    Hi, I am Mike you can call me on 222-222-XXX
    La la la la... November rain
    This is Tom, an Employee:
    Hi, I am Sam, I work at Things Ltd.. Call me on 444-222-XXX
    La la la la... Born to be wild
    Let's use a slice of Men and see what happens
    Hi, I am Paul you can call me on 111-222-XXX
    Hi, I am Sam, I work at Golang Inc.. Call me on 444-222-XXX
    Hi, I am Mike you can call me on 222-222-XXX
```

那么interface里面到底能存什么值呢？如果我们定义了一个interface的变量，那么这个变量里面可以存实现这个interface的任意类型的对象。例如上面例子中，我们定义了一个Men interface类型的变量m，那么m里面可以存Human、Student或者Employee值

> 当然，使用指针的方式，也是可以的
>
> 但是，接口对象不能调用实现对象的属性

## interface函数参数

interface的变量可以持有任意实现该interface类型的对象，这给我们编写函数(包括method)提供了一些额外的思考，可以通过定义interface参数，让函数接受各种类型的参数



# 接口类型

## 接口与鸭子类型

先直接来看维基百科里的定义：

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.

翻译过来就是：如果某个东西长得像鸭子，像鸭子一样游泳，像鸭子一样嘎嘎叫，那它就可以被看成是一只鸭子。

Duck Typing，鸭子类型，是动态编程语言的一种对象推断策略，它更关注对象能如何被使用，而不是对象的类型本身。Go 语言作为一门静态语言，它通过通过接口的方式完美支持鸭子类型。

而在静态语言如 Java, C++ 中，必须要显示地声明实现了某个接口，之后，才能用在任何需要这个接口的地方。如果你在程序中调用某个数，却传入了一个根本就没有实现另一个的类型，那在编译阶段就不会通过。这也是静态语言比动态语言更安全的原因。

Go 语言作为一门现代静态语言，是有后发优势的。它引入了动态语言的便利，同时又会进行静态语言的类型检查，写起来是非常 Happy 的。Go 采用了折中的做法：不要求类型显示地声明实现了某个接口，只要实现了相关的方法即可，编译器就能检测到。

总结一下，鸭子类型是一种动态语言的风格，在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由它"当前方法和属性的集合"决定。Go 作为一种静态语言，通过接口实现了鸭子类型，实际上是 Go 的编译器在其中作了隐匿的转换工作。

## go语言多态性

Go中的多态性是在接口的帮助下实现的。正如我们已经讨论过的，接口可以在Go中隐式地实现。如果类型为接口中声明的所有方法提供了定义，则实现一个接口。让我们看看在接口的帮助下如何实现多态。

任何定义接口所有方法的类型都被称为隐式地实现该接口。

类型接口的变量可以保存实现接口的任何值。接口的这个属性用于实现Go中的多态性。



# 接口断言

前面说过，因为空接口 interface{}没有定义任何函数，因此 Go 中所有类型都实现了空接口。当一个函数的形参是interface{}，那么在函数中，需要对形参进行断言，从而得到它的真实类型。

语法格式：

```go
// 安全类型断言

<目标类型的值>，<布尔参数> := <表达式>.( 目标类型 )

//非安全类型断言

<目标类型的值> := <表达式>.( 目标类型 )
```

示例代码：

```go
package main

import "fmt"

func main() {

   var i1 interface{} = new (Student)
   s := i1.(Student) //不安全，如果断言失败，会直接panic

   fmt.Println(s)

    var i2 interface{} = new(Student)
    s, ok := i2.(Student) //安全，断言失败，也不会panic，只是ok的值为false
    if ok {
        fmt.Println(s)
    }
}

type Student struct {

}
```

断言其实还有另一种形式，就是用在利用 switch语句判断接口的类型。每一个case会被顺序地考虑。当命中一个case 时，就会执行 case 中的语句，因此 case 语句的顺序是很重要的，因为很有可能会有多个 case匹配的情况。

```go
switch ins:=s.(type) {
    case Triangle:
        fmt.Println("三角形。。。",ins.a,ins.b,ins.c)
    case Circle:
        fmt.Println("圆形。。。。",ins.radius)
    case int:
        fmt.Println("整型数据。。")
    }
```



# 参考

> 1. Golang中国，https://www.qfgolang.com/