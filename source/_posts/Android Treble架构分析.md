---
title: Android Treble架构分析 
date: 2019-07-25 15:39:06
categories:
- Android
tags:
- Android
- Treble
typora-copy-images-to: Android Treble架构分析
typora-root-url: Android Treble架构分析
---

# Treble 简介

​	Android 8.0 版本的一项新元素是 Project Treble。这是 Android 操作系统框架在架构方面的一项重大改变，旨在让制造商以更低的成本更轻松、更快速地将设备更新到新版 Android 系统。Project Treble 适用于搭载 Android 8.0 及后续版本的所有新设备（这种新的架构已经在 Pixel 手机的开发者预览版中投入使用）。

## 系统更新

![](1-1-1.png)

​	Android 7.x 及更早版本中没有正式的供应商接口，因此设备制造商必须更新大量 Android 代码才能将设备更新到新版 Android 系统：



![](/1-1-2.png)

​	Treble 提供了一个稳定的新供应商接口，供设备制造商访问 Android 代码中特定于硬件的部分，这样一来，设备制造商只需更新 Android 操作系统框架，即可跳过芯片制造商直接提供新的 Android 版本。



## Android 经典框架

为了更好的了解Treble 架构里面的HAL，首先了解一下Android的经典架构。

![](/1-2-1.png)



​	在Android O之前，HAL是一个个的.so库，通过dlopen来进行打开，库和framework位于同一个进程。如图所示

![](/1-2-2.bmp)



## Treble架构

​	为了能够让Android O之前的版本升级到Android O，Android设计了Passthrough模式，经过转换，可以方便的使用已经存在代码，不需要重新编写相关的HAL。HIDL分为两种模式：Passthrough和Binderized。

* Binderized： Google官方翻译成绑定试HAL。
* Passthrough：Google官方翻译成直通式HAL。

大致框架图如下，对于Android O之前的设备，对应图1，对于从之前的设备升级到O的版本，对应图2、图3. 对于直接基于Android O开发的设备，对应图4。

![](/1-3-1.png)

这 ①②③④ 四种模式，是到目前为止四种实现架构。

* ① 是 Treble Project 之前使用的实现架构，使用的是传统 HAL 和旧版 HAL
* ② 直通模式，passthrough mode。如图所示，Framework 和 HAL 层工作在同一个进程当中，下面的 HAL 是使用 HIDL 封装后的库，是直通式 HAL。这些库文件也可用于 ③ 绑定模式
* ③ 绑定模式，binderized mode。是直通式 HAL binder 化，变为绑定式 HAL。Framework 和  HAL 层工作在不同的进程，之间通过 Binder 进行 IPC
* ④ 纯绑定式。相对于 ③ 来说，绑定式 HAL 中并不包含直通式 HAL，因此称为纯绑定式

新的架构之下，framework和hal运行在不同的进程，所有的HAL采用新的HIDL技术来完成。

![](/1-3-2.bmp)



