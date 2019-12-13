---
title: Go 数组和切片
date: 2019-12-13 15:42:36
categories:
- Go
tags:
- Go
typora-root-url: Go-数组和切片
typora-copy-images-to: Go-数组和切片
---

# 数组

Go 语言提供了数组类型的数据结构。
数组是具有相同唯一类型的一组已编号且长度固定的数据项序列，这种类型可以是任意的原始类型例如整形、字符串或者自定义类型。

数组元素可以通过索引（位置）来读取（或者修改），索引从0开始，第一个元素索引为 0，第二个索引为 1，以此类推。数组的下标取值范围是从0开始，到长度减1。

数组一旦定义后，大小不能更改。

## 声明和初始化数组

需要指明数组的大小和存储的数据类型。

```go
var variable_name [SIZE] variable_type
```

示例代码：

```go
var balance [10] float32
var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
var balance = []float32{1000.0, 2.0, 3.4, 7.0, 50.0}
var balance = [...] int{1,2,3,4,5}// 根据元素的个数，设置数组的大小
```

> 初始化数组中 {} 中的元素个数不能大于 [] 中的数字。
>
> 如果忽略 [] 中的数字不设置数组大小，Go 语言会根据元素的个数来设置数组的大小：

## 数组是值类型

Go中的数组是值类型，而不是引用类型。这意味着当它们被分配给一个新变量时，将把原始数组的副本分配给新变量。如果对新变量进行了更改，则不会在原始数组中反映。

```go
package main

import "fmt"

func main() {  
    a := [...]string{"USA", "China", "India", "Germany", "France"}
    b := a // a copy of a is assigned to b
    b[0] = "Singapore"
    fmt.Println("a is ", a)
    fmt.Println("b is ", b) 
}
```

运行结果：

```shell
a is [USA China India Germany France]  
b is [Singapore China India Germany France] 
```

> 数组的大小是类型的一部分。因此[5]int和[25]int是不同的类型。因此，数组不能被调整大小。不要担心这个限制，因为切片的存在是为了解决这个问题。

## 数组遍历

### 数组长度

通过将数组作为参数传递给len函数，可以获得数组的长度。

```go
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    fmt.Println("length of a is",len(a))

}
// length of a is 4
```

### 遍历数组

```go
package main

import "fmt"

func main() {  
    a := [...]float64{67.7, 89.8, 21, 78}
    for i := 0; i < len(a); i++ { //looping from 0 to the length of the array
        fmt.Printf("%d th element of a is %.2f\n", i, a[i])
    }
}
```

### for ... range 遍历

```go
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

如果您只需要值并希望忽略索引，那么可以通过使用_ blank标识符替换索引来实现这一点。

```go
for _, v := range a { //ignores index  
}
```



## 多为数组

Go 语言支持多维数组，以下为常用的多维数组声明语法方式：

```go
var variable_name [SIZE1][SIZE2]...[SIZEN] variable_type
var threedim [5][10][4]int
```

```go
a = [3][4]int{  
 {0, 1, 2, 3} ,   /*  第一行索引为 0 */
 {4, 5, 6, 7} ,   /*  第二行索引为 1 */
 {8, 9, 10, 11}   /*  第三行索引为 2 */
}
```



# 切片

Go 语言切片是对数组的抽象。

Go 数组的长度不可改变，在特定场景中这样的集合就不太适用，Go中提供了一种灵活，功能强悍的内置类型切片("动态数组"),与数组相比切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大

切片是一种方便、灵活且强大的包装器。切片本身没有任何数据。它们只是对现有数组的引用。

切片与数组相比，不需要设定长度，在[]中不用设定值，相对来说比较自由

从概念上面来说slice像一个结构体，这个结构体包含了三个元素：

1. 指针，指向数组中slice指定的开始位置
2. 长度，即slice的长度
3. 最大长度，也就是slice开始位置到数组的最后位置的长度

## 定义切片

切片不需要说明长度。

使用make()函数创建切片

```go
var slice1 []type = make([]type, len)
//也可以简写为
slice1 := make([]type, len)
```

```go
make([]T, length, capacity)
```

## 初始化切片

```go
s[0] = 1
s[1] = 2
s[2] = 3
```

```go
s :=[] int {1,2,3 } 
```

```go
s := arr[startIndex:endIndex] 
```

将arr中从下标startIndex到endIndex-1 下的元素创建为一个新的切片（**前闭后开**），长度为endIndex-startIndex

示例：

```go
package main

import (  
    "fmt"
)

func main() {  
    a := [5]int{76, 77, 78, 79, 80}
    var b []int = a[1:4] //creates a slice from a[1] to a[3]
    fmt.Println(b)
}
```

## 修改切片

slice没有自己的任何数据。它只是底层数组的一个表示。对slice所做的任何修改都将反映在底层数组中。

示例代码：

```go

import (  
    "fmt"
)

func main() {  
    darr := [...]int{57, 89, 90, 82, 100, 78, 67, 69, 59}
    dslice := darr[2:5]
    fmt.Println("array before",darr)
    for i := range dslice {
        dslice[i]++
    }
    fmt.Println("array after",darr) 
}
```

运行结果：

```shell
array before [57 89 90 82 100 78 67 69 59]  
array after [57 89 91 83 101 78 67 69 59]  
```

当多个片共享相同的底层数组时，每个元素所做的更改将在数组中反映出来。

示例代码：

```go
package main

import (  
    "fmt"
)

func main() {  
    numa := [3]int{78, 79 ,80}
    nums1 := numa[:] //creates a slice which contains all elements of the array
    nums2 := numa[:]
    fmt.Println("array before change 1",numa)
    nums1[0] = 100
    fmt.Println("array after modification to slice nums1", numa)
    nums2[1] = 101
    fmt.Println("array after modification to slice nums2", numa)
}
```

运行结果：

```shell
array before change 1 [78 79 80]  
array after modification to slice nums1 [100 79 80]  
array after modification to slice nums2 [100 101 80]  
```

## 切片长度和容量

切片的长度是切片中元素的数量。切片的容量是从创建切片的索引开始的底层数组中元素的数量。

切片是可索引的，并且可以由 len() 方法获取长度
切片提供了计算容量的方法 cap() 可以测量切片最长可以达到多少

```go
package main

import "fmt"

func main() {
   var numbers = make([]int,3,5)

   printSlice(numbers)
}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```

运行结果：

```shell
len=3 cap=5 slice=[0 0 0]
```

## 空切片

一个切片在未初始化之前默认为 nil，长度为 0

```go
package main

import "fmt"

func main() {
   var numbers []int

   printSlice(numbers)

   if(numbers == nil){
      fmt.Printf("切片是空的")
   }
}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```

## append()和copy()

append 向slice里面追加一个或者多个元素，然后返回一个和slice一样类型的slice
copy 函数copy从源slice的src中复制元素到目标dst，并且返回复制的元素的个数

append函数会改变slice所引用的数组的内容，从而影响到引用同一数组的其它slice。 但当slice中没有剩
余空间（即(cap-len) == 0）时，此时将动态分配新的数组空间。返回的slice数组指针将指向这个空间，而原
数组的内容将保持不变；其它引用此数组的slice则不受影响

下面的代码描述了从拷贝切片的 copy 方法和向切片追加新元素的 append 方法

```go
package main

import "fmt"

func main() {
   var numbers []int
   printSlice(numbers)

   /* 允许追加空切片 */
   numbers = append(numbers, 0)
   printSlice(numbers)

   /* 向切片添加一个元素 */
   numbers = append(numbers, 1)
   printSlice(numbers)

   /* 同时添加多个元素 */
   numbers = append(numbers, 2,3,4)
   printSlice(numbers)

   /* 创建切片 numbers1 是之前切片的两倍容量*/
   numbers1 := make([]int, len(numbers), (cap(numbers))*2)

   /* 拷贝 numbers 的内容到 numbers1 */
   copy(numbers1,numbers)
   printSlice(numbers1)   
}

func printSlice(x []int){
   fmt.Printf("len=%d cap=%d slice=%v\n",len(x),cap(x),x)
}
```

运行结果：

```shell
len=0 cap=0 slice=[]
len=1 cap=2 slice=[0]
len=2 cap=2 slice=[0 1]
len=5 cap=8 slice=[0 1 2 3 4]
len=5 cap=12 slice=[0 1 2 3 4]
```

> numbers1与numbers两者不存在联系，numbers发生变化时，numbers1是不会随着变化的。也就是说copy方法是不会建立两个切片的联系的



# 参考

> 1. Golang中国，https://www.qfgolang.com/