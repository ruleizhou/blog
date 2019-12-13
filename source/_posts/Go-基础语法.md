---
title: Go 基础语法
date: 2019-12-12 16:24:18
categories:
- Go
tags:
- Go
typora-root-url: Go-基础语法
typora-copy-images-to: Go-基础语法
---

# 变量

## 什么是变量

变量是为存储特定类型的值而提供给内存位置的名称。在go中声明变量有多种语法。

所以变量的本质就是一小块内存，用于存储数据，在程序运行过程中数值可以改变

## 声明变量

- 指定变量类型，声明后若不赋值，使用默认值

  ```go
  var name type
  name = value
  ```

- 根据值自行判定变量类型(类型推断Type inference)

  ```go
  var name = value
  ```

- ，省略var, 注意 :=左侧的变量不应该是已经声明过的(多个变量同时声明时，至少保证一个是新变量)，否则会导致编译错误(简短声明)

  ```go
  name := value
  
  // 例如
  var a int = 10
  var b = 10
  c : = 10
  ```

  > 这种方式它只能被用在函数体内，而不可以用于全局变量的声明与赋值

## 多变量声明

- 以逗号分隔，声明与赋值分开，若不赋值，存在默认值

  ```go
  var name1, name2, name3 type
  name1, name2, name3 = v1, v2, v3
  ```

- 直接赋值，下面的变量类型可以是不同的类型

  ```go
  var name1, name2, name3 = v1, v2, v3
  ```

- 集合类型

  ```go
  var (
      name1 type1
      name2 type2
  )
  ```

## 注意事项

- 变量必须先定义才能使用
- go语言是静态语言，要求变量的类型和赋值的类型必须一致。
- 变量名不能冲突。(同一个作用于域内不能冲突)
- 简短定义方式，左边的变量名至少有一个是新的
- 简短定义方式，不能定义全局变量。
- 变量的零值。也叫默认值。
- 变量定义了就要使用，否则无法通过编译。



# 常量

## 常量声明

常量是一个简单值的标识符，在程序运行时，不会被修改的量。

```go
显式类型定义： const b string = "abc"
隐式类型定义： const b = "abc"
```

常量的注意事项：

- 常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型
- 不曾使用的常量，在编译的时候，是不会报错的
- 显示指定类型的时候，必须确保常量左右值类型一致，需要时可做显示类型转换。这与变量就不一样了，变量是可以是不同的类型值

## iota

iota，特殊常量，可以认为是一个可以被编译器修改的常量

iota 可以被用作枚举值：

```go
// 每当 iota 在新的一行被使用时
const (
    a = iota
    b = iota
    c = iota
)
```

iota 示例

```go
package main

import "fmt"

func main() {
    const (
            a = iota   //0
            b          //1
            c          //2
            d = "ha"   //独立值，iota += 1
            e          //"ha"   iota += 1
            f = 100    //iota +=1
            g          //100  iota +=1
            h = iota   //7,恢复计数
            i          //8
    )
    fmt.Println(a,b,c,d,e,f,g,h,i)
}
```

运行结构

```shell
0 1 2 ha ha 100 100 7 8
```

如果中断iota自增，则必须显式恢复。且后续自增值按行序递增

自增默认是int类型，可以自行进行显示指定类型

数字常量不会分配存储空间，无须像变量那样通过内存寻址来取值，因此无法获取地址

# 数据类型

## 基础数据类型

![](/type.jpg)

### 布尔型bool

布尔型的值只可以是常量 true 或者 false。一个简单的例子：var b bool = true

### 数值型

1. 整型

   - int8

     有符号 8 位整型 (-128 到 127)

   - int16

     有符号 16 位整型 (-32768 到 32767)

   - int32

     有符号 32 位整型 (-2147483648 到 2147483647)

   - int64

     有符号 64 位整型 (-9223372036854775808 到 9223372036854775807)

   - uint8

     无符号 8 位整型 (0 到 255)

   - uint16

     无符号 16 位整型 (0 到 65535)

   - uint32

     无符号 32 位整型 (0 到 4294967295)

   - uint64

     无符号 64 位整型 (0 到 18446744073709551615)

   > int和uint:根据底层平台，表示32或64位整数。除非需要使用特定大小的整数，否则通常应该使用int来表示整数。
   > 大小:32位系统32位，64位系统64位。
   > 范围:-2147483648到2147483647的32位系统和-9223372036854775808到9223372036854775807的64位系统。

2. 浮点型

   - float32

     IEEE-754 32位浮点型数

   - float64

     IEEE-754 64位浮点型数

   - complex64

     32 位实数和虚数

   - complex128

     32 位实数和虚数

3. 其他

   - byte

     类似 uint8

   - rune

     类似 int32

   - uint

     32 或 64 位

   - int

     32 或 64 位

   - uintptr

     无符号整型，用于存放一个指针

### 字符串型

字符串就是一串固定长度的字符连接起来的字符序列。Go的字符串是由单个字节连接起来的。Go语言的字符串的字节使用UTF-8编码标识Unicode文本

```go
    var str string
    str = "Hello World"
```

### 数据类型转换

> 语法格式：Type(Value)
>
> 常数：在有需要的时候，会自动转型
>
> 变量：需要手动转型 T(V)
>
> 注意点：兼容类型可以转换

## 复合数据类型

1. 指针类型
2. 数组类型
3. 结构化类型
4. Channel类型
5. 函数类型
6. 切片类型
7. 接口类型
8. Map类型



# 运算符

## 算数运算符

```go
+
-
*
/
%(求余) 
++
--
```

## 关系运算符

```go
==
!= 
>
< 
>= 
<=
```

## 逻辑运算符

```go
&&	
||
!
```

## 位运算符

```go
&
|
^
&^
<<
>>
```

## 赋值运算符

```go
=
+=
-=
*=
/=
%=
<<=
>>=
&=
^=
|=
```

## 优先级运算符优先级

有些运算符拥有较高的优先级，二元运算符的运算方向均是从左至右。下表列出了所有运算符以及它们的优先级，由上至下代表优先级由高到低：

> | 优先级 | 运算符           |
> | :----- | :--------------- |
> | 7      | ~ ! ++ --        |
> | 6      | * / % << >> & &^ |
> | 5      | + - ^            |
> | 4      | == != < <= >= >  |
> | 3      | <-               |
> | 2      | &&               |
> | 1      | \|\|             |
>
> 当然，你可以通过使用括号来临时提升某个表达式的整体运算优先级。



# 键盘输入和打印输出

## 打印输出

### fmt包

fmt包实现了类似C语言printf和scanf的格式化I/O。格式化verb（’verb’）源自C语言但更简单。

详见官网fmt的API：https://golang.google.cn/pkg/fmt/

### 导入包

```go
import "fmt"
```

### 常用打印函数

- 打印

  [func Print(a …interface{}) (n int, err error)](https://golang.google.cn/pkg/fmt/#Print)

- 格式化打印

  [func Printf(format string, a …interface{}) (n int, err error)](https://golang.google.cn/pkg/fmt/#Printf)

  格式化打印中的常用占位符：

  > ```go
  > 格式化打印占位符：
  >             %v,原样输出
  >             %T，打印类型
  >             %t,bool类型
  >             %s，字符串
  >             %f，浮点
  >             %d，10进制的整数
  >             %b，2进制的整数
  >             %o，8进制
  >             %x，%X，16进制
  >                 %x：0-9，a-f
  >                 %X：0-9，A-F
  >             %c，打印字符
  >             %p，打印地址
  >             。。。
  > ```

- 打印后换行

  [func Println(a …interface{}) (n int, err error)](https://golang.google.cn/pkg/fmt/#Println)

## 键盘输入

### fmt包读取键盘输入

- [func Scan(a …interface{}) (n int, err error)](https://golang.google.cn/pkg/fmt/#Scan)

  > Scan 从标准输入中读取数据，并将数据用空白分割并解析后存入提供的变量中（换行符会被当作空白处理），变量必须以指针传入。当读到 EOF 或所有变量都填写完毕则停止扫描。返回成功解析的参数数量。

- [func Scanf(format string, a …interface{}) (n int, err error)](https://golang.google.cn/pkg/fmt/#Scanf)

  > Scanf 从标准输入中读取数据，并根据格式字符串 format 对数据进行解析,将解析结果存入参数 a 所提供的变量中，变量必须以指针传入。

- [func Scanln(a …interface{}) (n int, err error)](https://golang.google.cn/pkg/fmt/#Scanln)

  > Scanln 和 Scan 类似，只不过遇到换行符就停止扫描

### bufio包读取

https://golang.google.cn/pkg/bufio/

使用实例

```go
package main

import (
    "fmt"
    "os"
    "bufio"
)

func main() {
    fmt.Println("请输入一个字符串：")
    reader := bufio.NewReader(os.Stdin)
    s1, _ := reader.ReadString('\n')
    fmt.Println("读到的数据：", s1)

}
```



# 参考

> 1. Golang中国，https://www.qfgolang.com/