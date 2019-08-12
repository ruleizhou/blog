---
title: 初识FreeRTOS
date: 2019-08-12 15:46:56
categories:
- FreeRTOS
tags:
- FreeRTOS
typora-copy-images-to: 初识FreeRTOS
typora-root-url: 初识FreeRTOS
---

# FreeRTOS

​	FreeRTOS 由 美国 的 Richard Barry 于 2003 年 发布， Richard Barry 是 FreeRTOS 的 拥 有者 和 维护者， 在过 去的 十 多年 中 FreeRTOS 历经 了 9 个 版本， 与众 多半 导体 厂商 合作 密切， 有数 百万 开发者， 是 目前 市场 占有 率 最高 的 RTOS。

​	FreeRTOS 于 2018 年 被 亚马逊 收购， 改名 为 AWS FreeRTOS， 版本 号 升级 为 V10， 且 开源 协议 也由 原来 的 GPLv2+ 修改 为 MIT， 与 GPLv2+ 相比， MIT 更加 开放， 你 完全可以 理解 为 完全 免费。 V9 以前 的 版本 还是 维持 原样， V10 版本 相比 于 V9 就是 加入 了 一些 物 联网 相关 的 组件， 内核 基本 不变。



## FreeRTOS

​	FreeRTOS 是一 款“ 开源 免费” 的 实时 操作系统， 遵循 的 是 GPLv2+ 的 许可 协议。 这里 说到 的 开源， 指的 是 可以 免费 获取 FreeRTOS 的 源 代码， 且 当你 的 产品 使用 了 FreeRTOS 而 没有 修改 FreeRTOS 内核 源 码 时， 你的 产品 的 全部 代码 都可以 闭 源， 不用 开源， 但是 当你 修改 了 FreeRTOS 内核 源 码 时， 就必须 将 修改 的 这部 分 开源， 反馈 给 社区， 其他 应用 部分 不用 开源。 免费 的 意思是 无论 你是 个人 还是 公司， 都可以 免费 地 使用， 不需要 花费 一分钱。

## OpenRTOS

​	FreeRTOS 和 OpenRTOS 拥有 的 代码 是 一样 的， 但是 可从 官方 获取 的 服务 却是 不一样 的。 FreeRTOS 号称 免费， OpenRTOS 号称 收费。

| 比较的项目                         |         FreeRTOS         | OpenRTOS |
| :--------------------------------- | :----------------------: | :------: |
| 是否免费                           |            是            |    否    |
| 是否商业使用                       |            是            |    是    |
| 是否需要版权费                     |            否            |    否    |
| 是否提供技术支持                   |            否            |    是    |
| 是否被法律保护                     |            否            |    是    |
| 是否需要开源工程代码               |            否            |    否    |
| 是否需要开源修改的内核代码         |            是            |    否    |
| 是否需要声明产品使用了FreeRTOS     | 如果发布源码，则需要声明 |    否    |
| 是否需要提供FreeRTOS的整个工程代码 | 如果发布源码，则需要声明 |    否    |



## SaveRTOS

​	SaveRTOS 也 基于 FreeRTOS， 但是 SaveRTOS 为 某些 特定 的 领域 做了 安全 相关 的 设计， 有关 SaveRTOS 获得 的 安全 验证。 SaveRTOS 需要 收费。

| 行业类别 | 验证编号           |
| -------- | ------------------ |
| 工控     | IEC 61508          |
| 铁路     | EN 50128           |
| 医疗     | IEC 62304/FDA 510K |
| 核工业   | IEC 61513          |
| 汽车电子 | ISO 26262          |
| 加工业   | IEC 61511          |
| 航空     | DO178B             |



# FreeRTOS 资料获取

FreeRTOS的源码和相应的官方书籍均可从[官网](https://www.freertos.org/)获取。![](/1-1.png)

## 获取源码/书籍

*   官网![](/1-2.png)

    *   源码

        点击上图的`Download Source`按钮，可以下载FreeRTOS最新版本的源码。如果要下载以往版本可以从托管网站上下载。

    *   书籍

        点击上图的`PDF Books`按钮可以下载FreeRTOS官方的两本电子书

        *   API参考手册

            FreeRTOS V10. 0. 0 Reference Manual. pdf

        *   入门教程

            Mastering_ the_ FreeRTOS_ Real_ Time_ Kernel- A_ Hands- On_ Tutorial_ Guide. pdf

*   托管网站下载

    [下载网址](https://sourceforge.net/projects/freertos/files/FreeRTOS/)

# FreeRTOS的编程风格

## 数据类型

​	在 FreeRTOS 中， 使用 的 数据 类型 虽然 都是 标准 C 里面 的 数据 类型， 但是 针对 不同 的 处理器， 对 标准 C 的 数据 类型 又 进行 了 重 定义， 给 它们 设置 了 一个 新的 名字， 比如 为 char 重新 定义 了 一个 名字 portCHAR， 这里 的 port 表示 接口， 在 将 FreeRTOS 移植 到 处理器 上 时， 需要 用 这些 接口 文件 把 它们 连接 在一起。 但是 用户 在 写 程序 时并 非 一定 要 遵循 FreeRTOS 的 风格， 仍可 以 直接 用 C 语言 的 标准 类型。 在 FreeRTOS 中， int 型 从不 使用， 只 使用 short 型 和 long 型。 在 Cortex- M 内核 的 MCU 中， short 为 16 位， long 为 32 位。

​	FreeRTOS 中 详细 的 数据 类型 重 定义 在 portmacro. h 头 文件 中 实现。![](/1-3.png)

```c
/* Type definitions. */
#define portCHAR        char
#define portFLOAT       float
#define portDOUBLE      double
#define portLONG        long
#define portSHORT       short
#define portSTACK_TYPE  uint32_t
#define portBASE_TYPE   long

typedef portSTACK_TYPE StackType_t;
typedef long BaseType_t;
typedef unsigned long UBaseType_t;

#if( configUSE_16_BIT_TICKS == 1 )
    typedef uint16_t TickType_t;
    #define portMAX_DELAY ( TickType_t ) 0xffff
#else
    typedef uint32_t TickType_t;
    #define portMAX_DELAY ( TickType_t ) 0xffffffffUL

    /* 32-bit tick type on a 32-bit architecture, so reads of the tick count do
    not need to be guarded with a critical section. */
    #define portTICK_TYPE_IS_ATOMIC 1
#endif
```

​	在 编程 时，如果 用户 没有 明确 指定 char 的 符号 类型， 那么 编译器 会 默认 指定 char 型 的 变量 为 无符号 或者 有 符号。正是 基于 这个 原因，在 FreeRTOS 中， 我们 都 需要 明确 指定 变量 char 是有 符号 的 还是 无符号 的。

## 变量

​	在 FreeRTOS 中， 定义变量时往往会把 变量 的 类型当作前缀加在变量 上，这样做的好处是让用户 一看到这个变量就知道该变量的类型。比如char型变量的前缀是 c，short 型变量的前缀是s，long 型变量的前缀是 l， portBASE_ TYPE 类型变量的前缀是 x。 还有其他的数据类型，比如 数据 结构、任务句柄、队列 句柄等定义的变量名的前缀也是 x。

​	如果 一个变量是无符号型的， 那么会有 一个前缀 u， 如果是一 个 指针变量， 则会有 一个 前缀 p。 因此，当 我们定义一个无符号的 char 型变量时会加 一个 uc 前缀， 当定义 一个 char 型的指针变量时会加 一个 pc 前缀。

## 函数

​	函数名包含了函数返回值的类型、函数所在的文件名和函数的功能，如果是私有的函数， 则会加 一个 prv（ private）的前缀。 特别地，在 函数 名 中 加入了函数所在的文件名，这将 帮助用户提高寻找函数定义的效率并了解函数作用，具体 举例 如下：

*   vTaskPrioritySet() 函数 的 返回 值 为 void 型， 在 task. c 文件 中 定义。
*   xQueueReceive() 函数 的 返回 值 为 portBASE_ TYPE 型， 在 queue. c 文件 中 定义。

## 宏

宏军邮大写字母表示，并配有小写字母前缀，前缀用于表示该宏在哪个头文件定义。

| 前缀                             | 宏定义文件       |
| -------------------------------- | ---------------- |
| port(ex: portMax_DELAY)          | portable.h       |
| task(ex: taskENTER_CRITICAL())   | task.h           |
| pd(ex: pdTRUE)                   | projdefs.h       |
| config(ex: configUSE_PREEMPTION) | FreeRTOSConfig.h |
| err(ex: errUEUE_FULL)            | projdefs.h       |

注意：

*   信号量的函数都是一个宏定义。但是其函数的命名方法遵循函数的命名方法而不是宏定义的方法。

## 格式

​	1 个 Tab 键 等于 4 个 空格键。 我们 在 编程 时 最好 使用 空格键 而 不是 使用 Tab 键， 当 2 个 编译器 的 Tab 键 大小 设置 得不 一样 时， 移植 代码 时 格式 就会 变乱， 而使 用 空格键 不会 出现 这种 问题。