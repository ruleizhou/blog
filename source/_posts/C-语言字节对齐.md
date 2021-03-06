---
title: C 语言字节对齐
typora-root-url: C-语言字节对齐
date: 2019-09-13 16:25:38
categories:
- C/C++
tags:
- C/C++
---

&emsp;在阅读代码特别是C语法代码的过程中，其实经常会遇到#pragma pack(n)，#pragma pack()，#pragma pack(pack,n)，#pragma pack(pop,n)等，其中n是可以省略的。并且这些语句一般出现在结构体前面。我们一般都知道该语句是用来进行内存对齐的，关于字节对齐如下会进行详细介绍。

# 什么是字节对齐

&emsp;现代计算机中，内存空间按照字节划分，理论上可以从任何起始地址访问任意类型的变量。但实际中在访问特定类型变量时经常在特定的内存地址访问，这就需要各种类型数据按照一定的规则在空间上排列，而不是顺序一个接一个地存放，这就是对齐。

# 字节对齐的原因和作用

&emsp;不同硬件平台对存储空间的处理上存在很大的不同。某些平台对特定类型的数据只能从特定地址开始存取，而不允许其在内存中任意存放。例如Motorola 68000 处理器不允许16位的字存放在奇地址，否则会触发异常，因此在这种架构下编程必须保证字节对齐。

&emsp;但最常见的情况是，如果不按照平台要求对数据存放进行对齐，会带来存取效率上的损失。比如32位的Intel处理器通过总线访问(包括读和写)内存数据。每个总线周期从偶地址开始访问32位内存数据，内存数据以字节为单位存放。如果一个32位的数据没有存放在4字节整除的内存地址处，那么处理器就需要2个总线周期对其进行访问，显然访问效率下降很多。

&emsp;因此，通过合理的内存对齐可以提高访问效率。为使CPU能够对数据进行快速访问，数据的起始地址应具有“对齐”特性。比如4字节数据的起始地址应位于4字节边界上，即起始地址能够被4整除。

&emsp;此外，合理利用字节对齐还可以有效地节省存储空间。但要注意，在32位机中使用1字节或2字节对齐，反而会降低变量访问速度。因此需要考虑处理器类型。还应考虑编译器的类型。在VC/C++和GNU GCC中都是默认是4字节对齐。		

# 对齐的分类和准则

## 结构体对齐

&emsp;在C语言中，结构体是种复合数据类型，其构成元素既可以是基本数据类型(如int、long、float等)的变量，也可以是一些复合数据类型(如数组、结构体、联合等)的数据单元。编译器为结构体的每个成员按照其自然边界(alignment)分配空间。各成员按照它们被声明的顺序在内存中顺序存储，第一个成员的地址和整个结构的地址相同。

&emsp;字节对齐的问题主要就是针对结构体。

### 对齐准则

* 四个重要的节本概念：

  * 数据类型自身的对齐值

    char型数据自身对齐值为1字节，short型数据为2字节，int/float型为4字节，double型为8字节。

  * 结构体或类的自身对齐值

    其成员中自身对齐值最大的那个值。

  * 指定对齐值

    #pragma pack (value)时的指定对齐值value。

  * 数据成员、结构体和类的有效对齐值

    自身对齐值和指定对齐值中较小者，即有效对齐值=min{自身对齐值，当前指定的pack值}。

  基于上面这些值，就可以方便地讨论具体数据结构的成员和其自身的对齐方式。

  其中，有效对齐值N是最终用来决定数据存放地址方式的值。有效对齐N表示“对齐在N上”，即该数据的“存放起始地址%N=0”。而数据结构中的数据变量都是按定义的先后顺序存放。第一个数据变量的起始地址就是数据结构的起始地址。结构体的成员变量要对齐存放，结构体本身也要根据自身的有效对齐值圆整(即结构体成员变量占用总长度为结构体有效对齐值的整数倍)。

* 结构体字节对齐的细节和具体编译器实现相关，但一般而言满足三个准则：

  *  结构体变量的首地址能够被其最宽基本类型成员的大小所整除；
  * 结构体每个成员相对结构体首地址的偏移量(offset)都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节(internal adding)；
  * 结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要编译器会在最末一个成员之后加上填充字节{trailing padding}。

  对于以上规则的说明如下：

  1. 编译器在给结构体开辟空间时，首先找到结构体中最宽的基本数据类型，然后寻找内存地址能被该基本数据类型所整除的位置，作为结构体的首地址。将这个最宽的基本数据类型的大小作为上面介绍的对齐模数。
  2. 为结构体的一个成员开辟空间之前，编译器首先检查预开辟空间的首地址相对于结构体首地址的偏移是否是本成员大小的整数倍，若是，则存放本成员，反之，则在本成员和上一个成员之间填充一定的字节，以达到整数倍的要求，也就是将预开辟空间的首地址后移几个字节。
  3. 结构体总大小是包括填充字节，最后一个成员满足上面两条以外，还必须满足第三条，否则就必须在最后填充几个字节以达到本条要求。

  ```C
  /* OFFSET宏定义可取得指定结构体某成员在结构体内部的偏移 */
  #define OFFSET(st, field)     (size_t)&(((st*)0)->field)
  typedef struct{
      char  a;
      short b;
      char  c;
      int   d;
      char  e[3];
  }T_Test;
  
  int main(void){  
      printf("Size = %d\n  a-%d, b-%d, c-%d, d-%d\n  e[0]-%d, e[1]-%d, e[2]-%d\n",
             sizeof(T_Test), OFFSET(T_Test, a), OFFSET(T_Test, b),
             OFFSET(T_Test, c), OFFSET(T_Test, d), OFFSET(T_Test, e[0]),
             OFFSET(T_Test, e[1]),OFFSET(T_Test, e[2]));
      return 0;
  }
  ```

  执行后输出如下：

  ```shell
  Size = 16
  a-0, b-2, c-4, d-8
  e[0]-12, e[1]-13, e[2]-14
  ```

  具体分析如下：

  * 首先char a占用1个字节，没问题。
  * short b本身占用2个字节，根据上面准则2，需要在b和a之间填充1个字节。
  * char c占用1个字节，没问题。
  * int d本身占用4个字节，根据准则2，需要在d和c之间填充3个字节。
  * char e[3]；本身占用3个字节，根据原则3，需要在其后补充1个字节。
  * 因此，sizeof(T_Test) = 1 + 1 + 2 + 1 + 3 + 4 + 3 + 1 = 16字节。

### 字节对齐的隐患

#### 数据类型转换

代码中关于对齐的隐患，很多都是隐式的。如在强制类型的转换的时候：

```c
int main(void){  
    unsigned int i = 0x12345678;
        
    unsigned char *p = (unsigned char *)&i;
    *p = 0x00;
    unsigned short *p1 = (unsigned short *)(p+1);
    *p1 = 0x0000;

    return 0;
}
```

最后两句代码，从奇数边界去访问unsigned short型变量，显然不符合对齐的规定。在X86上，类似的操作只会影响效率；但在MIPS或者SPARC上可能导致error，因为它们要求必须字节对齐。

#### 处理器间数据通信

&emsp; 处理器间通过消息(对于C/C++而言就是结构体)进行通信时，需要注意字节对齐以及字节序的问题。

&emsp;大多数编译器提供内存对其的选项供用户使用。这样用户可以根据处理器的情况选择不同的字节对齐方式。例如C/C++编译器提供的#pragma pack(n) n=1，2，4等，让编译器在生成目标文件时，使内存数据按照指定的方式排布在1，2，4等字节整除的内存地址处。

&emsp;然而在不同编译平台或处理器上，字节对齐会造成消息结构长度的变化。编译器为了使字节对齐可能会对消息结构体进行填充，不同编译平台可能填充为不同的形式，大大增加处理器间数据通信的风险。

&emsp;下面以32位处理器为例，提出一种内存对齐方法以解决上述问题。

&emsp;对于本地使用的数据结构，为提高内存访问效率，采用四字节对齐方式；同时为了减少内存的开销，合理安排结构体成员的位置，减少四字节对齐导致的成员之间的空隙，降低内存开销。

&emsp;对于处理器之间的数据结构，需要保证消息长度不会因不同编译平台或处理器而导致消息结构体长度发生变化，使用一字节对齐方式对消息结构进行紧缩；为保证处理器之间的消息数据结构的内存访问效率，采用字节填充的方式自己对消息中成员进行四字节对齐。

&emsp;数据结构的成员位置要兼顾成员之间的关系、数据访问效率和空间利用率。顺序安排原则是：四字节的放在最前面，两字节的紧接最后一个四字节成员，一字节紧接最后一个两字节成员，填充字节放在最后。举例如下：

```c
typedef struct tag_T_MSG{
    long  ParaA;
    long  ParaB;
    short ParaC；
    char  ParaD;
    char  Pad;   //填充字节
}T_MSG;
```

#### 排查对齐问题

如果出现对齐或者赋值问题可查看:

* 编译器的字节序大小端设置；
* 处理器架构本身是否支持非对齐访问；
* 如果支持看设置对齐与否，如果没有则看访问时需要加某些特殊的修饰来标志其特殊访问操作。 

### 更改对齐方式

主要是更改C编译器的缺省字节对齐方式。

 在缺省情况下，C编译器为每一个变量或是数据单元按其自然对界条件分配空间。一般地，可以通过下面的方法来改变缺省的对界条件：

- 使用伪指令#pragma pack(n)：C编译器将按照n个字节对齐；
- 使用伪指令#pragma pack()： 取消自定义字节对齐方式。

## 栈内存对齐

在VC/C++中，栈的对齐方式不受结构体成员对齐选项的影响。总是保持对齐且对齐在4字节边界上。

```c
#pragma pack(push, 1)  //后面可改为1, 2, 4, 8
struct StrtE{
    char m1;
    long m2;
};
#pragma pack(pop)

int main(void){  
    char a;
    short b;
    int c;
    double d[2];
    struct StrtE s;
        
    printf("a    address:   %p\n", &a);
    printf("b    address:   %p\n", &b);
    printf("c    address:   %p\n", &c);
    printf("d[0] address:   %p\n", &(d[0]));
    printf("d[1] address:   %p\n", &(d[1]));
    printf("s    address:   %p\n", &s);
    printf("s.m2 address:   %p\n", &(s.m2));
    return 0;
}
```

结果如下：

```shell
a    address:   	0xbfc4cfff
b    address:   	0xbfc4cffc
c    address:   	0xbfc4cff8
d[0] address:    0xbfc4cfe8
d[1] address:    0xbfc4cff0
s    address:   	0xbfc4cfe3
s.m2 address:  0xbfc4cfe4
```

可以看出都是对齐到4字节。并且前面的char和short并没有被凑在一起(成4字节)，这和结构体内的处理是不同的。

## 位域对齐

### 位域定义

有些信息在存储时，并不需要占用一个完整的字节，而只需占几个或一个二进制位。例如在存放一个开关量时，只有0和1两种状态，用一位二进位即可。为了节省存储空间和处理简便，C语言提供了一种数据结构，称为“位域”或“位段”。

位域是一种特殊的结构成员或联合成员(即只能用在结构或联合中)，用于指定该成员在内存存储时所占用的位数，从而在机器内更紧凑地表示数据。每个位域有一个域名，允许在程序中按域名操作对应的位。这样就可用一个字节的二进制位域来表示几个不同的对象。

 位域定义与结构定义类似，其形式为：

```c
struct 位域结构名
{位域列表};
```

其中位域列表形式为：

```c
类型说明符位域名：位域长度
```

位域的使用和结构体成员使用相同，其一般形式为：

```c
位域变量名.位域名
```

 位域在本质上就是一种结构类型，不过其成员是按二进位分配的。位域变量的说明与结构变量说明的方式相同，可先定义后说明、同时定义说明或直接说明。 

 位域的使用主要为下面两种情况：

* 当机器可用内存空间较少而使用位域可大量节省内存时。如把结构作为大数组的元素时。
* 当需要把一结构体或联合映射成某预定的组织结构时。如需要访问字节内的特定位时。

### 对齐准则

位域成员不能单独被取sizeof值。下面主要讨论含有位域的结构体的sizeof。

C99规定int、unsigned int和bool可以作为位域类型，但编译器几乎都对此作了扩展，允许其它类型的存在。位域作为嵌入式系统中非常常见的一种编程工具，优点在于压缩程序的存储空间。

其对齐规则大致为：

1. 如果相邻位域字段的类型相同，且其位宽之和小于类型的sizeof大小，则后面的字段将紧邻前一个字段存储，直到不能容纳为止；
2. 如果相邻位域字段的类型相同，但其位宽之和大于类型的sizeof大小，则后面的字段将从新的存储单元开始，其偏移量为其类型大小的整数倍;
3. 如果相邻的位域字段的类型不同，则各编译器的具体实现有差异，VC6采取不压缩方式，Dev-C++和GCC采取压缩方式；
4. 如果位域字段之间穿插着非位域字段，则不进行压缩；
5. 整个结构体的总大小为最宽基本类型成员大小的整数倍，而位域则按照其最宽类型字节数对齐。

### 注意事项

关于位域操作有几点需要注意：

1. 位域的地址不能访问，因此不允许将&运算符用于位域。不能使用指向位域的指针也不能使用位域的数组(数组是种特殊指针)。

2. 位域不能作为函数返回的结果。

3. 位域以定义的类型为单位，且位域的长度不能够超过所定义类型的长度。

4. 位域可以不指定位域名，但不能访问无名的位域。只用作填充或调整位置，占位大小取决于该类型。

   ```c
    struct BitField3{
        char element1  : 3;
        char  :6;
        char element3  : 5;
    };
   ```

   结构体大小为3。因为element1占3位，后面要保留6位而char为8位，所以保留的6位只能放到第2个字节。同样element3只能放到第3字节。

   ```c
    struct BitField4{
        char element1  : 3;
        char  :0;
        char element3  : 5;
    };
   ```

   长度为0的位域告诉编译器将下一个位域放在一个存储单元的起始位置。如上，编译器会给成员element1分配3位，接着跳过余下的4位到下一个存储单元，然后给成员element3分配5位。故上面的结构体大小为2。

5. 位域的表示范围

   - 位域的赋值不能超过其可以表示的范围；
   - 位域的类型决定该编码能表示的值的结果。



# #pragma pack()

## 简述

在所有的预处理指令中，`#Pragma` 指令可能是最复杂的了，它的作用是设定编译器的状态或者是指示编译器完成一些特定的动作。`#pragma`指令对每个编译器给出了一个方法，在保持与C和C++语言完全兼容的情况下，给出主机或操作系统专有的特征。依据定义，编译指示是机器或操作系统专有的，且对于每个编译器都是不同的。

## 作用

`#pragma pack` 的主要作用就是改变编译器的内存对齐方式，这个指令在网络报文的处理中有着重要的作用，`#pragma pack(n)`是他最基本的用法，其作用是改变编译器的对齐方式， 不使用这条指令的情况下，编译器默认采取`#pragma pack(8)`也就是8字节的默认对齐方式，n值可以取（`1`， `2`， `4`， `8`， `16`） 中任意一值。

## 命令详解

### #pragma pack(show)

`#pragma pack(show)`显示当前内存对齐的字节数。也就是`packing aligment`。

```c
#pragma pack(show)
int main(){
	return 0;
}
```

*在程序中#pragma pack(show)会在编译阶段提出一个警告，说明当前对齐字节数。*

### #pragma pack(push\[,identifier\]\[,n\])

* 单纯使用`#pragma pack(push)`会将当前的对齐字节数压入栈顶，并设置这个值为新的对齐字节数， 就是说不会改变这个值。
* 使用`#pragma pack(push, n)` 会将当前的对齐字节数压入栈顶，并设置n为新的对齐字节数。
* `#pragma pack(push, identifier [, n])`会在上面的操作基础上为这个对齐字节数附上一个标识符， 这里注意这个标识符只能以（`$`、`_`、`字母`）开始， 标识符中可以有（`$`、`_`、`字母`、`数字`），并且标识符不能是关键字(`push， pop可以作为标识符`)。这个标识符的作用我会在pop中详细介绍。

### #pragma pack(pop\[,identifier\]\[,n\])

* 单纯使用`#pragma pack(pop)`会弹出栈顶对齐字节数，并设置其为新的内存对齐字节数。
* 使用`#pragma pack(pop, n)`情况就不同了， 他会弹出栈顶并直接丢弃，设置`n`为其新的内存对齐字节数。
* `#pragma pack(pop, identifier [, n])`较为复杂，编译器执行这条执行时会从栈顶向下顺序查找匹配的`identifier`，找到`identifier`相同的这个数之后将从栈顶到`identifier`，包括找到`identifier`全部`pop`弹出， 若没有找到则不进行任何操作。