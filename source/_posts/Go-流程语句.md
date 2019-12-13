---
title: Go 流程语句
date: 2019-12-13 14:34:35
categories:
- Go
tags:
- Go
typora-root-url: Go-流程语句
typora-copy-images-to: Go-流程语句
---

# if 分支语句

语法格式：

```go
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
}
```

```go
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
} else {
  /* 在布尔表达式为 false 时执行 */
}
```

```go
if 布尔表达式1 {
   /* 在布尔表达式1为 true 时执行 */
} else if 布尔表达式2{
   /* 在布尔表达式1为 false ,布尔表达式2为true时执行 */
} else{
   /* 在上面两个布尔表达式都为false时，执行*/
}
```

如果其中包含一个可选的语句组件(在评估条件之前执行)，则还有一个变体。它的语法格式如下：

```go
if statement; condition {  
}

if condition{

}
```

示例代码

```go
package main

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

> 需要注意的是，num的定义在if里，那么只能够在该if..else语句块中使用，否则编译器会报错的。

# switch分支语句

格式：

```go
switch var1 {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
}
```

## fallthrough

如需贯通后续的case，就添加fallthrough

```go
package main

import (
    "fmt"
)

type data [2]int

func main() {
    switch x := 5; x {
    default:
        fmt.Println(x)
    case 5:
        x += 10
        fmt.Println(x)
        fallthrough
    case 6:
        x += 20
        fmt.Println(x)
    }
}
```

case中的表达式是可选的，可以省略。如果该表达式被省略，则被认为是switch true，并且每个case表达式都被计算为true，并执行相应的代码块。

> switch的注意事项
>
> 1. case后的常量值不能重复
> 2. case后可以有多个常量值
> 3. fallthrough应该是某个case的最后一行。如果它出现在中间的某个地方，编译器就会抛出错误。

## Type Switch

switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。

```go
switch x.(type){
    case type:
       statement(s);      
    case type:
       statement(s); 
    /* 你可以定义任意个数的case */
    default: /* 可选 */
       statement(s);
}
```

```go
package main

import "fmt"

func main() {
   var x interface{}

   switch i := x.(type) {
      case nil:   
         fmt.Printf(" x 的类型 :%T",i)                
      case int:   
         fmt.Printf("x 是 int 型")                       
      case float64:
         fmt.Printf("x 是 float64 型")           
      case func(int) float64:
         fmt.Printf("x 是 func(int) 型")                      
      case bool, string:
         fmt.Printf("x 是 bool 或 string 型" )       
      default:
         fmt.Printf("未知型")     
   }   
}
```



# for 循环语句

语法结构：

```go
for init; condition; post { }
```

> 初始化语句只执行一次。在初始化循环之后，将检查该条件。如果条件计算为true，那么{}中的循环体将被执行，然后是post语句。post语句将在循环的每次成功迭代之后执行。在执行post语句之后，该条件将被重新检查。如果它是正确的，循环将继续执行，否则循环终止。
>
> 在for循环中声明的变量仅在循环范围内可用。

## for 循环变体

**所有的三个组成部分，即初始化、条件和post都是可选的。**

```go
// 效果与while相似
for condition { }
```

```go
// 效果与for(;;) 一样
for { }
```

for 循环的 range 格式可以对 slice、map、数组、字符串等进行迭代循环

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

## 跳出for 循环语句

- break：跳出循环体。break语句用于在结束其正常执行之前突然终止for循环
- continue：跳出一次循环。continue语句用于跳过for循环的当前迭代。在continue语句后面的for循环中的所有代码将不会在当前迭代中执行。循环将继续到下一个迭代。



# goto语句

goto：可以无条件地转移到过程中指定的行。

语法结构：

```go
goto label;
..
..
label: statement;
```

![](/goto1.jpg)



# 参考

> 1. Golang中国，https://www.qfgolang.com/