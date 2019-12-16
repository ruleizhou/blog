---
title: Go 函数
date: 2019-12-16 11:11:04
categories:
tags:
typora-root-url: Go-函数
typora-copy-images-to: Go-函数
---

# 函数

函数是执行特定任务的代码块。

语法格式：

```go
func funcName(parametername type1, parametername type2) (output1 type1, output2 type2) {
//这里是处理逻辑代码
//返回多个值
return value1, value2
}
```

> - func：函数由 func 开始声明
> - funcName：函数名称，函数名和参数列表一起构成了函数签名。
> - parametername type：参数列表，参数就像一个占位符，当函数被调用时，你可以将值传递给参数，这个值被称为实际参数。参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
> - output1 type1, output2 type2：返回类型，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types 不是必须的。
> - 上面返回值声明了两个变量output1和output2，如果你不想声明也可以，直接就两个类型。
> - 如果只有一个返回值且不声明返回值变量，那么你可以省略包括返回值的括号（即一个返回值可以不声明返回类型）
> - 函数体：函数定义的代码集合。

> go语言至少有一个main函数



# 函数的参数

## 参数的使用

- 形式参数

  定义函数时，用于接收外部传入的数据，叫做形式参数，简称形参。

- 实际参数

  调用函数时，传给形参的实际的数据，叫做实际参数，简称实参。

函数调用：

- 函数名称必须匹配
- 实参与形参必须一一对应：顺序，个数，类型

## 可变参数

Go函数支持变参。接受变参的函数是有着不定数量的参数的。为了做到这点，首先需要定义函数使其接受变参：

```go
func myfunc(arg ...int) {}
```

`arg ...int`告诉Go这个函数接受不定数量的参数。注意，这些参数的类型全部是int。在函数体中，变量arg是一个int的slice：

```go
for _, n := range arg {
fmt.Printf("And the number is: %d\n", n)
}
```

## 参数传递

go语言函数的参数也是存在**值传递**和**引用传递**

### 值传递

```go
package main

import (
   "fmt"
   "math"
)

func main(){
   /* 声明函数变量 */
   getSquareRoot := func(x float64) float64 {
      return math.Sqrt(x)
   }

   /* 使用函数 */
   fmt.Println(getSquareRoot(9))

}
```

### 引用传递

这就牵扯到了所谓的指针。我们知道，变量在内存中是存放于一定地址上的，修改变量实际是修改变量地址处的内存。

```go
package main
import "fmt"
//简单的一个函数，实现了参数+1的操作
func add1(a *int) int { // 请注意，
*a = *a+1 // 修改了a的值
return *a // 返回新值
} f
unc main() {
x := 3
fmt.Println("x = ", x) // 应该输出 "x = 3"
x1 := add1(&x) // 调用 add1(&x) 传x的地址
fmt.Println("x+1 = ", x1) // 应该输出 "x+1 = 4"
fmt.Println("x = ", x) // 应该输出 "x = 4"
}
```

- 传指针使得多个函数能操作同一个对象。
- 传指针比较轻量级 (8bytes),只是传内存地址，我们可以用指针传递体积大的结构体。如果用参数值传递的话, 在每次copy上面就会花费相对较多的系统开销（内存和时间）。所以当你要传递大的结构体的时候，用指针是一个明智的选择。
- **Go语言中slice，map这三种类型的实现机制类似指针**，所以可以直接传递，而不用取地址后传递指针。（注：若函数需改变slice的长度，则仍需要取地址传递指针）



# 函数的返回值

## 多个返回值

一个函数可以没有返回值，也可以有一个返回值，也可以有返回多个值。

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Mahesh", "Kumar")
   fmt.Println(a, b)
}
```

```go
func SumAndProduct(A, B int) (add int, Multiplied int) {
add = A+B
Multiplied = A*B
return
}
```

## 空白标识符

_是Go中的空白标识符。它可以代替任何类型的任何值。让我们看看这个空白标识符的用法。

比如rectProps函数返回的结果是面积和周长，如果我们只要面积，不要周长，就可以使用空白标识符。

```go
package main

import (  
    "fmt"
)

func rectProps(length, width float64) (float64, float64) {  
    var area = length * width
    var perimeter = (length + width) * 2
    return area, perimeter
}
func main() {  
    area, _ := rectProps(10.8, 5.6) // perimeter is discarded
    fmt.Printf("Area %f ", area)
}
```



# 函数中变量的作用域

## 局部变量

一个函数内部定义的变量，就叫做局部变量

变量在哪里定义，就只能在哪个范围使用，超出这个范围，我们认为变量就被销毁了。

## 全局变量

一个函数外部定义的变量，就叫做全局变量

所有的函数都可以使用，而且共享这一份数据



# defer

即延迟（defer）语句，延迟语句被用于执行一个函数调用，在这个函数之前，延迟语句返回。

## 延迟函数

你可以在函数中添加多个defer语句。当函数执行到最后时，这些defer语句会按照逆序执行，最后该函数返回。特别是当你在进行一些打开资源的操作时，遇到错误需要提前返回，在返回前你需要关闭相应的资源，不然很容易造成资源泄露等问题

- 如果有很多调用defer，那么defer是采用`后进先出`模式
- 在离开所在的方法时，执行（报错的时候也会执行）

```go
func ReadWrite() bool {
    file.Open("file")
    defer file.Close()
    if failureX {
          return false
    } i
    f failureY {
          return false
    } 
    return true
}
```

## 延迟方法

延迟并不仅仅局限于函数。延迟一个方法调用也是完全合法的。

```go
package main

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

运行结果：

```shell
Welcome John Smith 
```

## 延迟参数

延迟函数的参数在执行延迟语句时被执行，而不是在执行实际的函数调用时执行。

```go
package main

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

运行结果：

```shell
value of a before deferred function call 10  
value of a in deferred function 5 
```

## 堆栈的推迟

当一个函数有多个延迟调用时，它们被添加到一个堆栈中，并在Last In First Out（LIFO）后进先出的顺序中执行。

```go
package main

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

运行结果：

```shell
Orignal String: Naveen  
Reversed String: neevaN 
```



# 高阶函数

在Go语言中，函数也是一种数据类型。

## 匿名函数

匿名函数：没有名字的函数。

定义一个匿名函数，直接进行调用。通常只能使用一次。也可以使用匿名函数赋值给某个函数变量，那么就可以调用多次了。

```go
package main

import "fmt"

func main() {
    /*
    匿名：没有名字
        匿名函数：没有名字的函数。

    定义一个匿名函数，直接进行调用。通常只能使用一次。也可以使用匿名函数赋值给某个函数变量，那么就可以调用多次了。

    匿名函数：
        Go语言是支持函数式编程：
        1.将匿名函数作为另一个函数的参数，回调函数
        2.将匿名函数作为另一个函数的返回值，可以形成闭包结构。
     */
     fun1()
     fun1()
     fun2 := fun1
     fun2()

     //匿名函数
     func (){
        fmt.Println("我是一个匿名函数。。")
     }()

     fun3:=func(){
        fmt.Println("我也是一个匿名函数。。")
     }
     fun3()
     fun3()

     //定义带参数的匿名函数
     func (a,b int){
        fmt.Println(a,b)
     }(1,2)

     //定义带返回值的匿名函数
     res1 := func (a, b int)int{
        return a + b
     }(10,20) //匿名函数调用了，将执行结果给res1
     fmt.Println(res1)

     res2 := func (a,b int)int{
        return a + b
     } //将匿名函数的值，赋值给res2
     fmt.Println(res2)

     fmt.Println(res2(100,200))
}

func fun1(){
    fmt.Println("我是fun1()函数。。")
}

```

Go语言是支持函数式编程：

- 将匿名函数作为一个函数的参数，回调函数
- 将匿名函数作为一个函数的返回值，可以形成闭包结构。

## 回调函数

回调函数：callback，就是将一个函数fun2作为函数fun1的一个参数。那么fun2叫做回调函数，fun1叫做高阶函数。

```go
package main

import "fmt"

func main() {
    /*
    高阶函数：
        根据go语言的数据类型的特点，可以将一个函数作为另一个函数的参数。

    fun1(),fun2()
    将fun1函数作为了fun2这个函数的参数。

            fun2函数：就叫高阶函数
                接收了一个函数作为参数的函数，高阶函数

            fun1函数：回调函数
                作为另一个函数的参数的函数，叫做回调函数。
     */
    //设计一个函数，用于求两个整数的加减乘除运算
    fmt.Printf("%T\n", add)  //func(int, int) int
    fmt.Printf("%T\n", oper) //func(int, int, func(int, int) int) int

    res1 := add(1, 2)
    fmt.Println(res1)

    res2 := oper(10, 20, add)
    fmt.Println(res2)

    res3 := oper(5,2,sub)
    fmt.Println(res3)

    fun1:=func(a,b int)int{
        return a * b
    }

    res4:=oper(10,4,fun1)
    fmt.Println(res4)

    res5 := oper(100,8,func(a,b int)int{
        if b == 0{
            fmt.Println("除数不能为零")
            return 0
        }
        return a / b
    })
    fmt.Println(res5)

}
func oper(a, b int, fun func(int, int) int) int {
    fmt.Println(a, b, fun) //打印3个参数
    res := fun(a, b)
    return res
}

//加法运算
func add(a, b int) int {
    return a + b
}

//减法
func sub(a, b int) int {
    return a - b
}

```

## 闭包

闭包(closure)：一个外层函数中，有内层函数，该内层函数中，会操作外层函数的局部变量(外层函数中的参数，或者外层函数中直接定义的变量)，并且该外层函数的返回值就是这个内层函数。

 这个内层函数和外层函数的局部变量，统称为闭包结构。

```go
package main

import "fmt"

func main() {
    /*
    go语言支持函数式编程：
        支持将一个函数作为另一个函数的参数，
        也支持将一个函数作为另一个函数的返回值。

    闭包(closure)：
        一个外层函数中，有内层函数，该内层函数中，会操作外层函数的局部变量(外层函数中的参数，或者外层函数中直接定义的变量)，并且该外层函数的返回值就是这个内层函数。

        这个内层函数和外层函数的局部变量，统称为闭包结构。

        局部变量的生命周期会发生改变，正常的局部变量随着函数调用而创建，随着函数的结束而销毁。
        但是闭包结构中的外层函数的局部变量并不会随着外层函数的结束而销毁，因为内层函数还要继续使用。

     */
     res1 := increment() //res1 = fun
     fmt.Printf("%T\n",res1) //func() int
     fmt.Println(res1)
     v1 := res1()
     fmt.Println(v1) //1
     v2 := res1()
     fmt.Println(v2) //2
     fmt.Println(res1())
     fmt.Println(res1())
     fmt.Println(res1())
     fmt.Println(res1())

     res2 := increment()
     fmt.Println(res2)
     v3 :=res2()
     fmt.Println(v3) //1
     fmt.Println(res2())

     fmt.Println(res1())
}

func increment()func()int{ //外层函数
    //1.定义了一个局部变量
    i := 0
    //2.定义了一个匿名函数，给变量自增并返回
    fun := func ()int{ //内层函数
        i++
        return i
    }
    //3.返回该匿名函数
    return fun
}
```

闭包结构的注意点：局部变量的生命周期会发生改变，正常的局部变量随着函数调用而创建，随着函数的结束而销毁。但是闭包结构中的外层函数的局部变量并不会随着外层函数的结束而销毁，因为内层函数还要继续使用。



# 参考

> 1. Golang中国，https://www.qfgolang.com/