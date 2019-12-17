---
title: Go type
date: 2019-12-17 14:43:50
categories:
- Go
tags:
- Go
typora-root-url: Go-type
typora-copy-images-to: Go-type
---

type是go语法里的重要而且常用的关键字，type绝不只是对应于C/C++中的typedef。搞清楚type的使用，就容易理解go语言中的核心概念struct、interface、函数等的使用。

# 类型定义

## 定义结构体

使用type 可以定义结构体类型：

```go
//1、定义结构体
//结构体定义
type person struct {
   name string //注意后面不能有逗号
   age  int
}
```

## 定义接口

使用type 可以定义接口类型：

```go
type USB interface {
    start()
    end()
}
```

## 定义其他新的类型

使用type，还可以定义新类型。语法：

```go
type 类型名 Type
```

```go
package main

import "fmt"

type myint int
type mystr string

func main() {

     var i1 myint
     var i2 = 100
     i1 = 100
     fmt.Println(i1)
     //i1 = i2 //cannot use i2 (type int) as type myint in assignment
     fmt.Println(i1,i2)

     var name mystr
     name = "王二狗"
     var s1 string
     s1 = "李小花"
     fmt.Println(name)
     fmt.Println(s1)
     name = s1 //cannot use s1 (type string) as type mystr in assignment
}

```

## 定义函数类型

Go语言支持函数式编程，可以使用高阶编程语法。一个函数可以作为另一个函数的参数，也可以作为另一个函数的返回值，那么在定义这个高阶函数的时候，如果函数的类型比较复杂，我们可以使用type来定义这个函数的类型：

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {

     res1 := fun1()
     fmt.Println(res1(10,20))
}

type my_fun  func (int,int)(string)

//fun1()函数的返回值是my_func类型
func fun1 () my_fun{
    fun := func(a,b int) string {
        s := strconv.Itoa(a) + strconv.Itoa(b)
        return s
    }
    return fun
}

```



# 类型别名

类型别名的写法为：

```go
type 别名 = Type
```

类型别名规定：TypeAlias 只是 Type 的别名，本质上 TypeAlias 与 Type 是同一个类型。就像一个孩子小时候有小名、乳名，上学后用学名，英语老师又会给他起英文名，但这些名字都指的是他本人。

类型别名是 Go 1.9 版本添加的新功能。主要用于代码升级、迁移中类型的兼容性问题。在 C/C++语言中，代码重构升级可以使用宏快速定义新的一段代码。Go 语言中没有选择加入宏，而是将解决重构中最麻烦的类型名变更问题。

在 Go 1.9 版本之前的内建类型定义的代码是这样写的：

```go
type byte uint8
type rune int32
```

而在 Go 1.9 版本之后变为：

```go
type byte = uint8
type rune = int32
```



# 非本地类型不能定义方法

```go
package main
import (
    "time"
)
// 定义time.Duration的别名为MyDuration
type MyDuration = time.Duration
// 为MyDuration添加一个函数
func (m MyDuration) EasySet(a string) { //cannot define new methods on non-local type time.Duration
}
func main() {
}
```

以上代码报错。报错信息：cannot define new methods on non-local type time.Duration

编译器提示：不能在一个非本地的类型 time.Duration 上定义新方法。非本地方法指的就是使用 time.Duration 的代码所在的包，也就是 main 包。因为 time.Duration 是在 time 包中定义的，在 main 包中使用。time.Duration 包与 main 包不在同一个包中，因此不能为不在一个包中的类型定义方法。



# 参考

> 1. Golang中国，https://www.qfgolang.com/