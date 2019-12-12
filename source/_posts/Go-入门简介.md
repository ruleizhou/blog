---

title: Go 入门简介
date: 2019-12-11 15:48:14
categories:
- Go
tags:
- Go
typora-root-url: Go-入门简介
typora-copy-images-to: Go-入门简介
---

# Go 语言的核心特性

Go语言，作为编程语言的后生，站在巨人的肩膀上，吸收了其他一些编程语言的特点。

Go 编程语言是一个开源项目，它使程序员更具生产力。Go 语言具有很强的表达能力，它简洁、清晰而高效。得益于其并发机制， 用它编写的程序能够非常有效地利用多核与联网的计算机，其新颖的类型系统则使程序结构变得灵活而模块化。 Go 代码编译成机器码不仅非常迅速，还具有方便的垃圾收集机制和强大的运行时反射机制。 它是一个快速的、静态类型的编译型语言，感觉却像动态类型的解释型语言。(摘取自官网)。



## 思想

- Less can be more
- 大道至简,小而蕴真
- 让事情变得复杂很容易，让事情变得简单才难
- 深刻的工程文化



## 核心特定

Go语言之所以厉害，是因为它在服务端的开发中，总能抓住程序员的痛点，以最直接、简单、高效、稳定的方式来解决问题。这里我们并不会深入讨论GO语言的具体语法，只会将语言中关键的、对简化编程具有重要意义的方面介绍给大家，体验Go的核心特性。

### 并发编程

Go语言在并发编程方面比绝大多数语言要简洁不少，这一点是其最大亮点之一，也是其在未来进入高并发高性能场景的重要筹码。![](/bingfa1.jpg)

不同于传统的多进程或多线程，golang的并发执行单元是一种称为goroutine的协程。

由于在共享数据场景中会用到锁，再加上GC，其并发性能有时不如异步复用IO模型，因此相对于大多数语言来说，golang的并发编程简单比并发性能更具卖点。

在当今这个多核时代，并发编程的意义不言而喻。当然，很多语言都支持多线程、多进程编程，但遗憾的是，实现和控制起来并不是那么令人感觉轻松和愉悦。Golang不同的是，语言级别支持协程(goroutine)并发（协程又称微线程，比线程更轻量、开销更小，性能更高），操作起来非常简单，语言级别提供关键字（go）用于启动协程，并且在同一台机器上可以启动成千上万个协程。协程经常被理解为轻量级线程，一个线程可以包含多个协程，共享堆不共享栈。协程间一般由应用程序显式实现调度，上下文切换无需下到内核层，高效不少。协程间一般不做同步通讯，而golang中实现协程间通讯有两种：1）共享内存型，即使用全局变量+mutex锁来实现数据共享；2）消息传递型，即使用一种独有的channel机制进行异步通讯。

对比JAVA的多线程和GO的协程实现，明显更直接、简单。这就是GO的魅力所在，以简单、高效的方式解决问题，关键字go，或许就是GO语言最重要的标志。

**高并发是Golang语言最大的亮点**

### 内存回收(GC)

从C到C++，从程序性能的角度来考虑，这两种语言允许程序员自己管理内存，包括内存的申请和释放等。因为没有垃圾回收机制所以C/C++运行起来速度很快，但是随着而来的是程序员对内存使用上的很谨小慎微的考虑。因为哪怕一点不小心就可能会导致“内存泄露”使得资源浪费或者“野指针”使得程序崩溃等，尽管C++11后来使用了智能指针的概念，但是程序员仍然需要很小心的使用。后来为了提高程序开发的速度以及程序的健壮性，java和C#等高级语言引入了GC机制，即程序员不需要再考虑内存的回收等，而是由语言特性提供垃圾回收器来回收内存。但是随之而来的可能是程序运行效率的降低。

GC过程是：先stop the world，扫描所有对象判活，把可回收对象在一段bitmap区中标记下来，接着立即start the world，恢复服务，同时起一个专门gorountine回收内存到空闲list中以备复用，不物理释放。物理释放由专门线程定期来执行。

GC瓶颈在于每次都要扫描所有对象来判活，待收集的对象数目越多，速度越慢。一个经验值是扫描10w个对象需要花费1ms，所以尽量使用对象少的方案，比如我们同时考虑链表、map、slice、数组来进行存储，链表和map每个元素都是一个对象，而slice或数组是一个对象，因此slice或数组有利于GC。

GC性能可能随着版本不断更新会不断优化，这块没仔细调研，团队中有HotSpot开发者，应该会借鉴jvm gc的设计思想，比如分代回收、safepoint等。

-  内存自动回收，再也不需要开发人员管理内存
- 开发人员专注业务实现，降低了心智负担
- 只需要new分配内存，不需要释放

### 内存分配

初始化阶段直接分配一块大内存区域，大内存被切分成各个大小等级的块，放入不同的空闲list中，对象分配空间时从空闲list中取出大小合适的内存块。内存回收时，会把不用的内存重放回空闲list。空闲内存会按照一定策略合并，以减少碎片。

### 编译

编译涉及到两个问题：编译速度和依赖管理

目前Golang具有两种编译器，一种是建立在GCC基础上的Gccgo，另外一种是分别针对64位x64和32位x86计算机的一套编译器(6g和8g)。

依赖管理方面，由于golang绝大多数第三方开源库都在github上，在代码的import中加上对应的github路径就可以使用了，库会默认下载到工程的pkg目录下。

另外，编译时会默认检查代码中所有实体的使用情况，凡是没使用到的package或变量，都会编译不通过。这是golang挺严谨的一面。

### 网络编程

由于golang诞生在互联网时代，因此它天生具备了去中心化、分布式等特性，具体表现之一就是提供了丰富便捷的网络编程接口，比如socket用net.Dial(基于tcp/udp，封装了传统的connect、listen、accept等接口)、http用http.Get/Post()、rpc用client.Call(‘class_name.method_name’, args, &reply)，等等。

> 高性能HTTP Server

### 函数多返回值

在C，C++中，包括其他的一些高级语言是不支持多个函数返回值的。但是这项功能又确实是需要的，所以在C语言中一般通过将返回值定义成一个结构体，或者通过函数的参数引用的形式进行返回。而在Go语言中，作为一种新型的语言，目标定位为强大的语言当然不能放弃对这一需求的满足，所以支持函数多返回值是必须的。

函数定义时可以在入参后面再加(a,b,c)，表示将有3个返回值a、b、c。这个特性在很多语言都有，比如python。

这个语法糖特性是有现实意义的，比如我们经常会要求接口返回一个三元组（errno,errmsg,data），在大多数只允许一个返回值的语言中，我们只能将三元组放入一个map或数组中返回，接收方还要写代码来检查返回值中包含了三元组，如果允许多返回值，则直接在函数定义层面上就做了强制，使代码更简洁安全。

### 语言交互性

语言交互性指的是本语言是否能和其他语言交互，比如可以调用其他语言编译的库。

在Go语言中直接重用了大部份的C模块，这里称为Cgo.Cgo允许开发者混合编写C语言代码，然后Cgo工具可以将这些混合的C代码提取并生成对于C功能的调用包装代码。开发者基本上可以完全忽略这个Go语言和C语言的边界是如何跨越的。

golang可以和C程序交互，但不能和C++交互。可以有两种替代方案：1）先将c++编译成动态库，再由go调用一段c代码，c代码通过dlfcn库动态调用动态库（记得export LD_LIBRARY_PATH）；2）使用swig

### 异常处理

golang不支持try…catch这样的结构化的异常解决方式，因为觉得会增加代码量，且会被滥用，不管多小的异常都抛出。golang提倡的异常处理方式是：

-  普通异常：被调用方返回error对象，调用方判断error对象
- 严重异常：指的是中断性panic（比如除0），使用defer…recover…panic机制来捕获处理。严重异常一般由golang内部自动抛出，不需要用户主动抛出，避免传统try…catch写得到处都是的情况。当然，用户也可以使用panic(‘xxxx’)主动抛出，只是这样就使这一套机制退化成结构化异常机制了。

### 其他一些有趣的特性

- 类型推导：类型定义：支持`var abc = 10`这样的语法，让golang看上去有点像动态类型语言，但golang实际上时强类型的，前面的定义会被自动推导出是int类型。

  > 作为强类型语言，隐式的类型转换是不被允许的，记住一条原则：让所有的东西都是显式的。
  >
  > 简单来说，Go是一门写起来像动态语言，有着动态语言开发效率的静态语言。

- 一个类型只要实现了某个interface的所有方法，即可实现该interface，无需显式去继承。

  > Go编程规范推荐每个Interface只提供一到两个的方法。这样使得每个接口的目的非常清晰。另外Go的隐式推导也使得我们组织程序架构的时候更加灵活。在写JAVA／C++程序的时候，我们一开始就需要把父类／子类／接口设计好，因为一旦后面有变更，修改起来会非常痛苦。而Go不一样，当你在实现的过程中发现某些方法可以抽象成接口的时候，你直接定义好这个接口就OK了，其他代码不需要做任何修改，编译器的自动推导会帮你做好一切。

- 不能循环引用：即如果a.go中import了b，则b.go要是import a会报import cycle not allowed。好处是可以避免一些潜在的编程危险，比如a中的func1()调用了b中的func2()，如果func2()也能调用func1()，将会导致无限循环调用下去。

- defer机制：在Go语言中，提供关键字defer，可以通过该关键字指定需要延迟执行的逻辑体，即在函数体return前或出现panic时执行。这种机制非常适合善后逻辑处理，比如可以尽早避免可能出现的资源泄漏问题。

  可以说，defer是继goroutine和channel之后的另一个非常重要、实用的语言特性，对defer的引入，在很大程度上可以简化编程，并且在语言描述上显得更为自然，极大的增强了代码的可读性。

- “包”的概念：和python一样，把相同功能的代码放到一个目录，称之为包。包可以被其他包引用。main包是用来生成可执行文件，每个程序只有一个main包。包的主要用途是提高代码的可复用性。通过package可以引入其他包。

- 编程规范：GO语言的编程规范强制集成在语言中，比如明确规定花括号摆放位置，强制要求一行一句，不允许导入没有使用的包，不允许定义没有使用的变量，提供gofmt工具强制格式化代码等等。奇怪的是，这些也引起了很多程序员的不满，有人发表GO语言的XX条罪状，里面就不乏对编程规范的指责。要知道，从工程管理的角度，任何一个开发团队都会对特定语言制定特定的编程规范，特别像Google这样的公司，更是如此。GO的设计者们认为，与其将规范写在文档里，还不如强制集成在语言里，这样更直接，更有利用团队协作和工程管理。

- 交叉编译：比如说你可以在运行 Linux 系统的计算机上开发运行 Windows 下运行的应用程序。这是第一门完全支持 UTF-8 的编程语言，这不仅体现在它可以处理使用 UTF-8 编码的字符串，就连它的源码文件格式都是使用的 UTF-8 编码。Go 语言做到了真正的国际化！



## 功能

此处我们说个小段子：

很久以前，有一个IT公司，这公司有个传统，允许员工拥有20%自由时间来开发实验性项目。在2007的某一天，公司的几个大牛，正在用c++开发一些比较繁琐但是核心的工作，主要包括庞大的分布式集群，大牛觉得很闹心，后来c++委员会来他们公司演讲，说c++将要添加大概35种新特性。这几个大牛的其中一个人，名为：Rob Pike，听后心中一万个xxx飘过，“c++特性还不够多吗？简化c++应该更有成就感吧”。于是乎，Rob Pike和其他几个大牛讨论了一下，怎么解决这个问题，过了一会，Rob Pike说要不我们自己搞个语言吧，名字叫“go”，非常简短，容易拼写。其他几位大牛就说好啊，然后他们找了块白板，在上面写下希望能有哪些功能。接下来的时间里，大牛们开心的讨论设计这门语言的特性，经过漫长的岁月，他们决定，以c语言为原型，以及借鉴其他语言的一些特性，来解放程序员，解放自己，然后在2009年，go语言诞生。

以下就是这些大牛所罗列出的Go要有的功能：

- 规范的语法（不需要符号表来解析）
- 垃圾回收（独有）
- 无头文件
- 明确的依赖
- 无循环依赖
- 常量只能是数字
- int和int32是两种类型
- 字母大小写设置可见性（letter case sets visibility）
- 任何类型（type）都有方法（不是类型）
- 没有子类型继承（不是子类）
- 包级别初始化以及明确的初始化顺序
- 文件被编译到一个包里
- 包package-level globals presented in any order
- 没有数值类型转换（常量起辅助作用）
- 接口隐式实现（没有“implement”声明）
- 嵌入（不会提升到超类）
- 方法按照函数声明（没有特别的位置要求）
- 方法即函数
- 接口只有方法（没有数据）
- 方法通过名字匹配（而非类型）
- 没有构造函数和析构函数
- postincrement（如++i）是状态，不是表达式
- 没有preincrement(i++)和predecrement
- 赋值不是表达式
- 明确赋值和函数调用中的计算顺序（没有“sequence point”）
- 没有指针运算
- 内存一直以零值初始化
- 局部变量取值合法
- 方法中没有“this”
- 分段的堆栈
- 没有静态和其它类型的注释
- 没有模板
- 内建string、slice和map
- 数组边界检查

> ## 大牛真身
>
> 最大牌的当属B和C语言设计者、Unix和Plan 9创始人、1983年图灵奖获得者Ken Thompson，这份名单中还包括了Unix核心成员Rob Pike（go语言之父）、java HotSpot虚拟机和js v8引擎的开发者Robert Griesemer、Memcached作者Brad Fitzpatrick，等等。



# Go语言环境搭建

## Golang官方网站

Golang官方网站：https://golang.org/ 

![](/Golang.jpg)

> 因为Google和中国的关系，直接登录Golang的官网，需要翻墙

当然你也可以登录Golang的国内网站：https://golang.google.cn/

![](/Goland_cn.jpg)

## 下载

在Mac、Windows和Linux三个平台上都支持Golang。您可以从https://golang.org/dl/下载相应平台的安装包。

![](/go_download.jpg)

- Mac OS

  > 从https://golang.org/dl/下载osx安装程序。双击启动安装。按照提示，这应该在/usr/local/go中安装了Golang，并且还会将文件夹/usr/local/go/bin添加到您的PATH环境变量中。

- Windows

  > 从https://golang.org/dl/下载MSI安装程序。双击启动安装并遵循提示。这将在位置c中安装Golang:\Go，并且还将添加目录c:\Go\bin到您的path环境变量。

- Linux

  > 从https://golang.org/dl/下载tar文件，并将其解压到/usr/local。将/usr/local/go/bin添加到PATH环境变量中。

## 安装和配置环境

### Linux系统安装和配置

- 解压到指定目录

  ```shell
  sudo tar -xzf go1.xxx.tar.gz -C /usr/local
  ```

- 配置GOROOT

  ```shell
  # Go的安装目录
  export GOROOT="/usr/local/go"
  ```

  

- 配置GOPATH

  ```shell
  # Go项目代码存放的路径
  # 对于Ubuntu系统，默认使用Home/go目录作为GOPATH
  export GOPATH=$HOME/go
  ```

  > GO代码必须在工作空间内，工作空间是一个目录，其中包含三个子目录
  >
  > src	  --> 里面每一个子目录就是一个包。包内是Go源码文件
  >
  > pkg	--> 编译生成的，包的目标文件
  >
  > bin	 --> 生成的可执行文件

- 配置GOBIN

  ```shell
  # GOBIN需要添加到PATH中
  export GOBIN=$GOROOT/bin
  export PATH=$PATH:$GOBIN
  ```

- 测试安装

  - 版本检测

    ```shell
    go version
    ```

  - 配置信息

    ```shell
    go env
    ```



# Go源码文件

![](/yuanmawenjian1.png)

## 命令源码文件

声明自己属于 main 代码包、包含无参数声明和结果声明的 main 函数。

命令源码文件被安装以后，GOPATH 如果只有一个工作区，那么相应的可执行文件会被存放当前工作区的 bin 文件夹下；如果有多个工作区，就会安装到 GOBIN 指向的目录下。

命令源码文件是 Go 程序的入口。

同一个代码包中最好也不要放多个命令源码文件。多个命令源码文件虽然可以分开单独 go run 运行起来，但是无法通过 go build 和 go install。

## 库源码文件

库源码文件就是不具备命令源码文件上述两个特征的源码文件。存在于某个代码包中的普通的源码文件。

库源码文件被安装后，相应的归档文件（.a 文件）会被存放到当前工作区的 pkg 的平台相关目录下。

## 测试源码文件

名称以 _test.go 为后缀的代码文件，并且必须包含 Test 或者 Benchmark 名称前缀的函数：

```go
func TestXXX( t *testing.T) {

}
```

名称以 Test 为名称前缀的函数，只能接受 *testing.T 的参数，这种测试函数是功能测试函数。

```go
func BenchmarkXXX( b *testing.B) {

}
```

名称以 Benchmark 为名称前缀的函数，只能接受 *testing.B 的参数，这种测试函数是性能测试函数。



# GO 命令

目前go的基本命令有如下17个

| bug      | start a bug report                             |
| -------- | ---------------------------------------------- |
| build    | compile packages and dependencies              |
| clean    | remove object files and cached files           |
| doc      | show documentation for package or symbol       |
| env      | print Go environment information               |
| fix      | update packages to use new APIs                |
| fmt      | gofmt (reformat) package sources               |
| generate | generate Go files by processing source         |
| get      | download and install packages and dependencies |
| install  | compile and install packages and dependencies  |
| list     | list packages or modules                       |
| mod      | module maintenance                             |
| run      | compile and run Go program                     |
| test     | test packages                                  |
| tool     | run specified go tool                          |
| version  | print Go version                               |
| vet      | report likely mistakes in packages             |

其中和编译相关的有build、get、install、run这4个。接下来就依次看看这四个的作用。

在详细分析这4个命令之前，先罗列一下通用的命令标记，以下这些命令都可适用的：

| -a    | 用于强制重新编译所有涉及的 Go 语言代码包（包括 Go 语言标准库中的代码包），即使它们已经是最新的了。该标记可以让我们有机会通过改动底层的代码包做一些实验。 |
| ----- | :----------------------------------------------------------- |
| -n    | 使命令仅打印其执行过程中用到的所有命令，而不去真正执行它们。如果不只想查看或者验证命令的执行过程，而不想改变任何东西，使用它正好合适。 |
| -race | 用于检测并报告指定 Go 语言程序中存在的数据竞争问题。当用 Go 语言编写并发程序的时候，这是很重要的检测手段之一。 |
| -v    | 用于打印命令执行过程中涉及的代码包。这一定包括我们指定的目标代码包，并且有时还会包括该代码包直接或间接依赖的那些代码包。这会让你知道哪些代码包被执行过了。 |
| -work | 用于打印命令执行时生成和使用的临时工作目录的名字，且命令执行完成后不删除它。这个目录下的文件可能会对你有用，也可以从侧面了解命令的执行过程。如果不添加此标记，那么临时工作目录会在命令执行完毕前删除。 |
| -x    | 使命令打印其执行过程中用到的所有命令，并同时执行它们。       |

## go run

专门用来运行命令源码文件的命令，**注意，这个命令不是用来运行所有 Go 的源码文件的！**

go run 命令只能接受一个命令源码文件以及若干个库源码文件（必须同属于 main 包）作为文件参数，且**不能接受测试源码文件**。它在执行时会检查源码文件的类型。如果参数中有多个或者没有命令源码文件，那么 go run 命令就只会打印错误提示信息并退出，而不会继续执行。

## go build

go build 命令主要是用于测试编译。在包的编译过程中，若有必要，会同时编译与之相关联的包。

1. 如果是普通包，当你执行go build命令后，不会产生任何文件。
2. 如果是main包，当只执行go build命令后，会在当前目录下生成一个可执行文件。如果需要在$GOPATH/bin目录下生成相应的exe文件，需要执行go install 或者使用 go build -o 路径/可执行文件。
3. 如果某个文件夹下有多个文件，而你只想编译其中某一个文件，可以在 go build 之后加上文件名，例如 go build a.go；go build 命令默认会编译当前目录下的所有go文件。
4. 你也可以指定编译输出的文件名。比如，我们可以指定go build -o 可执行文件名，默认情况是你的package名(非main包)，或者是第一个源文件的文件名(main包)。
5. go build 会忽略目录下以”_”或者”.”开头的go文件。
6. 如果你的源代码针对不同的操作系统需要不同的处理，那么你可以根据不同的操作系统后缀来命名文件。

当代码包中有且仅有一个命令源码文件的时候，在文件夹所在目录中执行 go build 命令，会在该目录下生成一个与目录同名的可执行文件。

![](/go_build_1.jpg)

![](/go_build_2.png)

## go install

go install 命令是用来编译并安装代码包或者源码文件的。

go install 命令在内部实际上分成了两步操作：第一步是生成结果文件(可执行文件或者.a包)，第二步会把编译好的结果移到` $GOPATH/pkg `或者`$GOPATH/bin`。

可执行文件： 一般是 go install 带main函数的go文件产生的，有函数入口，所有可以直接运行。

.a应用包： 一般是 go install 不包含main函数的go文件产生的，没有函数入口，只能被调用。

go install 用于编译并安装指定的代码包及它们的依赖包。当指定的代码包的依赖包还没有被编译和安装时，该命令会先去处理依赖包。与 go build 命令一样，传给 go install 命令的代码包参数应该以导入路径的形式提供。并且，go build 命令的绝大多数标记也都可以用于.实际上，go install 命令只比 go build 命令多做了一件事，即：安装编译后的结果文件到指定目录。

安装代码包会在当前工作区的 pkg 的平台相关目录下生成归档文件（即 .a 文件）。

安装命令源码文件会在当前工作区的 bin 目录（如果 GOPATH 下有多个工作区，就会放在 GOBIN 目录下）生成可执行文件。

![](/go_install.jpg)

## go get

go get 命令用于从远程代码仓库（比如 Github ）上下载并安装代码包。**注意，go get 命令会把当前的代码包下载到 $GOPATH 中的第一个工作区的 src 目录中，并安装。**

> 使用 go get 下载第三方包的时候，依旧会下载到 $GOPATH 的第一个工作空间，而非 vendor 目录。当前工作链中并没有真正意义上的包依赖管理，不过好在有不少第三方工具可选。

go get 常用的一些标记如下：

| -d        | 让命令程序只执行下载动作，而不执行安装动作。                 |
| --------- | ------------------------------------------------------------ |
| -f        | 仅在使用`-u`标记时才有效。该标记会让命令程序忽略掉对已下载代码包的导入路径的检查。如果下载并安装的代码包所属的项目是你从别人那里 Fork 过来的，那么这样做就尤为重要了。 |
| -fix      | 让命令程序在下载代码包后先执行修正动作，而后再进行编译和安装。 |
| -insecure | 允许命令程序使用非安全的 scheme（如 HTTP ）去下载指定的代码包。如果你用的代码仓库（如公司内部的 Gitlab ）没有HTTPS 支持，可以添加此标记。请在确定安全的情况下使用它。 |
| -t        | 让命令程序同时下载并安装指定的代码包中的测试源码文件中依赖的代码包。 |
| -u        | 让命令利用网络来更新已有代码包及其依赖包。默认情况下，该命令只会从网络上下载本地不存在的代码包，而不会更新已有的代码包。 |

![](/go_get.jpg)

## go clean 

go clean 命令是用来移除当前源码包里面编译生成的文件，这些文件包括

- _obj/ 旧的object目录，由Makefiles遗留
- _test/ 旧的test目录，由Makefiles遗留
- _testmain.go 旧的gotest文件，由Makefiles遗留
- test.out 旧的test记录，由Makefiles遗留
- build.out 旧的test记录，由Makefiles遗留
- *.[568ao] object文件，由Makefiles遗留
- DIR(.exe) 由 go build 产生
- DIR.test(.exe) 由 go test -c 产生
- MAINFILE(.exe) 由 go build MAINFILE.go产生 

## go fmt

go fmt 命令主要是用来帮你格式化所写好的代码文件。

## go test

go test 命令，会自动读取源码目录下面名为*_test.go的文件，生成并运行测试用的可执行文件。默认的情况下，不需要任何的参数，它会自动把你源码包下面所有test文件测试完毕，当然你也可以带上参数，详情请参考go help testflag

## go doc

go doc 命令其实就是一个很强大的文档工具。

如何查看相应package的文档呢？ 例如builtin包，那么执行go doc builtin；如果是http包，那么执行go doc net/http；查看某一个包里面的函数，那么执行go doc fmt Printf；也可以查看相应的代码，执行go doc -src fmt Printf；

```shell
# 在浏览器中打开127.0.0.1:8080
godoc -http=:8080
```

## go fix

 用来修复以前老版本的代码到新版本，例如go1之前老版本的代码转化到go1

## go version

查看go当前的版本

## go env

查看当前go的环境变量

## go list

列出当前全部安装的package



# GO 开发工具

## Goland

### 安装

- 下载

  > [官网下载](https://www.jetbrains.com/go/download/#section=linux)

- 解压

  ```shell
  # 解压到相应目录
  tar -zxzf goland-2018.1.1.tar.gz  -C /opt
  ```

- 安装

  ```shell
  touch  $HOME/.local/share/applications/goland.desktop
  ```

  > ```shell
  > [Desktop Entry]
  > Type=Application
  > Name=GoLand
  > Icon=[GoLand的目录下的bin目录]/goland.png
  > Exec=[GoLand的目录下的bin目录]/goland.sh
  > Terminal=false
  > Categories=Application;
  > ```

- 永久注册及汉化

  [教程](https://github.com/ruleizhou/tools/tree/master/golang)

### 使用

- 创建项目
  - 新建GO项目![](/goland_1.bmp)
  - 项目配置![](/goland_2.png)
  - 配置GOROOT![](/goland_3.png)
  - 配置GOPATH![](/goland_4.png)

- 快捷键

  - 文件相关

    ```shell
    CTRL+E，					打开最近浏览过的文件。
    CTRL+SHIFT+E，	打开最近更改的文件。
    CTRL+N，					可以快速打开struct结构体。
    CTRL+SHIFT+N，	可以快速打开文件。
    ```

  - 代码格式化

    ```shell
    CTRL+ALT+T，		可以把代码包在一个块内，例如if{…}else{…}。
    CTRL+ALT+L，		格式化代码。
    CTRL+空格，		代码提示。
    CTRL+/,					单行注释。CTRL+SHIFT+/，进行多行注释。
    CTRL+B，				快速打开光标处的结构体或方法（跳转到定义处）。
    CTRL+“+/-”，	可以将当前方法进行展开或折叠。
    ```

  - 查找和定位

    ```shell
    CTRL+R，					替换文本。
    CTRL+F，					查找文本。
    CTRL+SHIFT+F，	进行全局查找。
    CTRL+G，					快速定位到某行。
    ```

  - 代码编辑

    ```shell
    ALT+Q，									  可以看到当前方法的声明。
    CTRL+Backspace，				 按单词进行删除。
    SHIFT+ENTER，					 可以向下插入新行，即使光标在当前行的中间。
    CTRL+X，									 删除当前光标所在行。
    CTRL+D,										复制当前光标所在行。
    ALT+SHIFT+UP/DOWN,		  可以将光标所在行的代码上下移动。
    CTRL+SHIFT+U，					可以将选中内容进行大小写转化。
    ```

## Liteide

### 安装

使用如下shell 来执行安装过程

```shell
if [ -d $linux_env/linux/tool/liteide ]; then
    echo "liteide already install"
else
    echo "install liteide"
    ## Git clone and build liteide ##
    sudo apt-get install qt5-default --fix-missing -y
    git clone https://github.com/visualfc/liteide.git
    cd liteide/build
    ./update_pkg.sh
    ./build_linux_qt5.sh

    ## Deploy it: ##
    ./deploy_linux_x64_qt5.sh
    
    ## copy to linux/tool/liteide ##
    mv liteide $linux_env/linux/tool/liteide
    cd -
    rm liteide -rf    
fi

if [ -f ~/.local/share/applications/liteide.desktop ]; then
    echo "liteide.desktop already exists"
else
    ## create desktop ##
    touch ~/.local/share/applications/liteide.desktop
    echo "[Desktop Entry]" >> ~/.local/share/applications/liteide.desktop
    echo "Type=Application" >> ~/.local/share/applications/liteide.desktop
    echo "Name=liteide" >> ~/.local/share/applications/liteide.desktop
    echo "Icon=$linux_env/linux/tool/liteide/share/liteide/welcome/images/liteide400.png" >> ~/.local/share/applications/liteide.desktop
    echo "Exec=$linux_env/linux/tool/liteide/bin/liteide" >> ~/.local/share/applications/liteide.desktop
    echo "Comment=LiteIDE is a simple, open source, cross-platform Go IDE." >> ~/.local/share/applications/liteide.desktop
    echo "Terminal=false" >> ~/.local/share/applications/liteide.desktop
    echo "Categories=Development;IDE;" >> ~/.local/share/applications/liteide.desktop
    echo "Name[zh_CN]=liteide" >> ~/.local/share/applications/liteide.desktop
fi
```



# 编码规范

## 命名规范

Go在命名时以字母a到Z或a到Z或下划线开头，后面跟着零或更多的字母、下划线和数字(0到9)。Go不允许在命名时中使用@、$和%等标点符号。Go是一种区分大小写的编程语言。

> 1. 当命名（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就**可以被外部包的代码所使用**（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；
> 2. **命名如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的**（像面向对象语言中的 private ）

### 包命名

保持package的名字和目录保持一致，尽量采取有意义的包名，简短，有意义，尽量和标准库不要冲突。包名应该为**小写**单词，不要使用下划线或者混合大小写。

```go
package demo

package main
```

### 文件命名

尽量采取有意义的文件名，简短，有意义，应该为**小写**单词，使用**下划线**分隔各个单词。

```go
my_test.go
```

### 结构体命名

- 采用驼峰命名法，首字母根据访问控制大写或者小写
- struct 申明和初始化格式采用多行，例如下面：

```go
// 多行申明
type User struct{
    Username  string
    Email     string
}

// 多行初始化
u := User{
    Username: "astaxie",
    Email:    "astaxie@gmail.com",
}
```

### 接口命名

- 命名规则基本和上面的结构体类型
- 单个函数的结构名以 “er” 作为后缀，例如 Reader , Writer 。

```go
type Reader interface {
        Read(p []byte) (n int, err error)
}
```

### 变量命名

- 和结构体类似，变量名称一般遵循驼峰法，首字母根据访问控制原则大写或者小写，但遇到特有名词时，需要遵循以下规则：
  - 如果变量为私有，且特有名词为首个单词，则使用小写，如 apiClient
  - 其它情况都应当使用该名词原有的写法，如 APIClient、repoID、UserID
  - 错误示例：UrlArray，应该写成 urlArray 或者 URLArray

- 若变量类型为 bool 类型，则名称应以 Has, Is, Can 或 Allow 开头

  ```go
  var isExist bool
  var hasConflict bool
  var canManage bool
  var allowGitHook bool
  ```

### 常量命名

- 常量均需使用全部大写字母组成，并使用下划线分词

  ```go
  	const APP_VER = "1.0"
  ```

- 如果是枚举类型的常量，需要先创建相应类型：

  ```go
  type Scheme string
  
  const (
      HTTP  Scheme = "http"
      HTTPS Scheme = "https"
  )
  ```

### 关键字

下面的列表显示了Go中的保留字。这些保留字不能用作常量或变量或任何其他标识符名称。

| break    | default     | func   | interface | select |
| -------- | ----------- | ------ | --------- | ------ |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

## 注释规范

Go提供C风格的`/* */`块注释和C ++风格的`//`行注释。行注释是常态；块注释主要显示为包注释

- 单行注释是最常见的注释形式，你可以在任何地方使用以 // 开头的单行注释
- 多行注释也叫块注释，均已以 / *开头，并以* / 结尾，且不可以嵌套使用，多行注释一般用于包的文档描述或注释成块的代码片段

go 语言自带的 godoc 工具可以根据注释生成文档，生成可以自动生成对应的网站，注释的质量决定了生成的文档的质量。每个包都应该有一个包注释，在package子句之前有一个块注释。对于多文件包，包注释只需要存在于一个文件中，任何一个都可以。

### 包注释

每个包都应该有一个包注释，一个位于package子句之前的块注释或行注释。包如果有多个go文件，只需要出现在一个go文件中（一般是和包同名的文件）即可。 包注释应该包含下面基本信息(请严格按照这个顺序，简介，创建人，创建时间）：

- 包的基本简介（包名，简介）
- 创建者，格式： 创建人： rtx 名
- 创建时间，格式：创建时间： yyyyMMdd

```shell
// util 包， 该包包含了项目共用的一些常量，封装了项目中一些共用函数。
// 创建人： han
// 创建时间： 20191212
```

### 结构(接口)注释

每个自定义的结构体或者接口都应该有注释说明，该注释对结构进行简要介绍，放在结构体定义的前一行，格式为： 结构体名， 结构体说明。同时结构体内的每个成员变量都要有说明，该说明放在成员变量的后面（注意对齐），实例如下：

```go
// User ， 用户对象，定义了用户的基础信息
type User struct{
    Username  string 		  // 用户名
    Email     		string 			// 邮箱
}
```

### 函数(方法)注释

每个函数，或者方法（结构体或者接口下的函数称为方法）都应该有注释说明，函数的注释应该包括三个方面（严格按照此顺序撰写）：

- 简要说明，格式说明：以函数名开头，“，”分隔说明部分
- 参数列表：每行一个参数，参数名开头，“，”分隔说明部分
- 返回值： 每行一个返回值

```go
// NewtAttrModel ， 属性数据层操作类的工厂方法
// 参数：
//      ctx ： 上下文信息
// 返回值：
//      属性操作类指针
func NewAttrModel(ctx *common.Context) *AttrModel {
}
```

### 代码逻辑注释

对于一些关键位置的代码逻辑，或者局部较为复杂的逻辑，需要有相应的逻辑说明，方便其他开发者阅读该段代码。

```go
// 从 Redis 中批量读取属性，对于没有读取到的 id ， 记录到一个数组里面，准备从 DB 中读取
xxxxx
xxxxxxx
xxxxxxx
```

### 注释风格

统一使用中文注释，对于中英文字符之间严格使用空格分隔， 这个不仅仅是中文和英文之间，英文和中文标点之间也都要使用空格分隔。

```go
// 从 Redis 中批量读取属性，对于没有读取到的 id ， 记录到一个数组里面，准备从 DB 中读取
```

上面 Redis 、 id 、 DB 和其他中文字符之间都是用了空格分隔。

- 建议全部使用单行注释
- 和代码的规范一样，单行注释不要过长，禁止超过 120 字符。

## 代码风格

### 缩进和折行

- 缩进直接使用 gofmt 工具格式化即可（gofmt 是使用 tab 缩进的）；
- 折行方面，一行最长不超过120个字符，超过的请使用换行展示，尽量保持格式优雅。

### 语句的结尾

Go语言中是不需要类似于Java需要冒号结尾，默认一行就是一条数据

如果你打算将多个语句写在同一行，它们则必须使用 **;**

### 括号和空格

括号和空格方面，也可以直接使用 gofmt 工具格式化（go 会强制左大括号不换行，换行会报语法错误），所有的运算符和操作数之间要留空格。

```go
// 正确的方式
if a > 0 {

} 

// 错误的方式
if a>0  // a ，0 和 > 之间应该空格
{       // 左大括号不可以换行，会报语法错误

}
```

### import规范

import在多行的情况下，goimports会自动帮你格式化，但是我们这里还是规范一下import的一些规范，如果你在一个文件里面引入了一个package，还是建议采用如下格式：

```go
import (
    "fmt"
)
```

如果你的包引入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下方式进行组织你的包：

```shell
import (
    "encoding/json"
    "strings"

    "myproject/models"
    "myproject/controller"
    "myproject/utils"

    "github.com/astaxie/beego"
    "github.com/go-sql-driver/mysql"
)   
```

有顺序的引入包，不同的类型采用空格分离，第一种实标准库，第二是项目包，第三是第三方包。

在项目中不要使用相对路径引入包：

```go
// 这是不好的导入
import “../net”

// 这是正确的做法
import “github.com/repo/proj/src/net”
```

但是如果是引入本项目中的其他包，最好使用相对路径。

### 错误处理

- 错误处理的原则就是不能丢弃任何有返回err的调用，不要使用 _ 丢弃，必须全部处理。接收到错误，要么返回err，或者使用log记录下来
- 尽早return：一旦有错误发生，马上返回
- 尽量不要使用panic，除非你知道你在做什么
- 错误描述如果是英文必须为小写，不需要标点结尾
- 采用独立的错误流进行处理

```go
if err != nil {
    // error handling
} else {
    // normal code
}

// 正确写法
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

### 测试

单元测试文件名命名规范为 example_test.go
测试用例的函数名称必须以 Test 开头，例如：TestExample
每个重要的函数都要首先编写测试用例，测试用例和正规代码一起提交方便进行回归测试

## 常用工具

### gofmt

大部分的格式问题可以通过gofmt解决， gofmt 自动格式化代码，保证所有的 go 代码与官方推荐的格式保持一致，于是所有格式有关问题，都以 gofmt 的结果为准。

### goimport

我们强烈建议使用 goimport ，该工具在 gofmt 的基础上增加了自动删除和引入包.

```shell
go get golang.org/x/tools/cmd/goimports
```

### go vet

vet工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前return的逻辑，struct的tag是否符合标准等。

```shell
go get golang.org/x/tools/cmd/vet
```



# 参考

> 1. Golang中国，https://www.qfgolang.com/