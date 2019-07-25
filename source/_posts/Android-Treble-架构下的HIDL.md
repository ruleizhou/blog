---
title: Android Treble 架构下的HIDL
date: 2019-07-25 22:34:01
categories:
- Android
- Treble
tags:
- Android
- Treble
typora-copy-images-to: Android-Treble-架构下的HIDL
typora-root-url: Android-Treble-架构下的HIDL
---

# HIDL 简介

​	HIDL 读作 hide-l，全称是 Hardware Interface Definition Language。它在 Android Project Treble 中被起草，在 Android 8.0 中被全面使用，其诞生目的是使 Android 可以在不重新编译 HAL 的情况下对 Framework 进行 OTA 升级。 

​	`使用 HIDL 描述的 HAL 描述文件替换旧的用头文件描述的 HAL 文件的过程称为 HAL 的 binder 化（binderization）`。所有运行 Android O 的设备都必须只支持 binder 化后的 HAL 模块。

​	HIDL是一种接口定义语言，描述了HAL和它的用户之间的接口。同aidi类似，我们只需要为hal定义相关接口，然后通过hidl-gen工具即可自动编译生成对应的C++或者java源文件，定义hal接口的文件命名为xxx.hal，为了编译这些.hal文件，需要编写相应的Android.bp或者Android.mk文件

-   Android.bp文件用于编译C++
-   Android.mk文件用于编译Java



## HIDL HAL 类型

### 直通模式(Passthrough)

​	和 Framework 工作在同一个进程当中，因没有 service 进程，服务也没有被事先注册到 
hwservicemanager，Framework 通过 Binder 得到的是同一个进程中的实例。在 manifest.xml 中 transport 类型为 passthrough。以 `android.hardware.graphics.composer@2.1` 为例，在 [Android.bp](https://android.googlesource.com/platform/hardware/interfaces/+/refs/tags/android-8.1.0_r65/graphics/composer/2.1/default/Android.bp#22) 中，Hwc.cpp 被编译为 `android.hardware.graphics.composer@2.1-impl.so`，其 [HIDL_FETCH_IComposer](https://android.googlesource.com/platform/hardware/interfaces/+/refs/tags/android-8.1.0_r65/graphics/composer/2.1/default/Hwc.cpp#747) 方法中会使用 hw_get_module() 方法去加载传统 HAL。

### 绑定模式(Binderized)

​	而将直通模式中的直通式 HAL，添加上 service 进程，修改 transport 类型为 hwbinder，就成为了绑定模式。service 的注册使用的是 [defaultPassthroughServiceImplementation()](https://android.googlesource.com/platform/hardware/interfaces/+/refs/tags/android-8.1.0_r65/graphics/composer/2.1/default/service.cpp#43) 方法。

### 纯绑定模式

​	service 的注册方法都是 `registerAsService()`，在 manifest.xml 中的 transport 类型为 hwbinder，不再单独编译 `*-impl.so`，而是全编译进 service 中。以 `android.hardware.power@1.1` 默认实现为例。`android.hardware.power@1.1-service` 服务启动时，服务的注册方法是 [registerAsService()](https://android.googlesource.com/platform/hardware/interfaces/+/62cc79bdf0c52c773602d9e93bbf732b1c54b934/power/1.1/default/service.cpp#74)。在 [Android.bp](https://android.googlesource.com/platform/hardware/interfaces/+/62cc79bdf0c52c773602d9e93bbf732b1c54b934/power/1.1/default/Android.bp#21) 中将两个源文件编译为 `android.hardware.power@1.1-service`。有意思的是在 service 的 main 方法中居然会去 [hw_get_module(POWER_HARDWARE_MODULE_ID, &hw_module)](https://android.googlesource.com/platform/hardware/interfaces/+/62cc79bdf0c52c773602d9e93bbf732b1c54b934/power/1.1/default/service.cpp#47) 加载传统 HAL，那这个默认实现 1.1 版相对于 1.0 版就没有意义了。也因并没有厂商使用这个默认版，[谷歌干脆移除了](https://android.googlesource.com/platform/hardware/interfaces/+/4497a5fe338c4a19dc31312641b2caa8454eb24e)。



## HIDL 文件的组织结构

​	每个 HIDL package包里都含有一个名为types.hal的文件，该文件中定义了这个包里所有 interface 共享的用户自定义数据类型，并且一般也会导入需要用到的其它包里的数据类型。

 	当前包中新的定义的 interface 可以继承自从其它包里导入的 interface，这样的继承关系可以使用extend关键字实现。

​	由 Google 提供的包叫做core package，包名始终以android.hardware.开头，以子系统名加以区分。比如 NFC 包的名字就应该为android.hardware.nfc，摄像头包的名字就应该为android.hardware.camera。这些 core包存放于hardware/interfaces/目录下。

​	由各芯片厂商和 ODM厂商提供的包叫做non-core package，包名形式一般以vendor.$(vendorName).hardware.开头，比如vendor.samsung.hardware.。这些non-core包一般存放于vendor/$(vendorName)/interfaces/目录下。 

​	包的版本使用主、次版本号进行描述，紧随包名之后。比如android.hardware.audio@2.0表述这个 audio 包的版本是 2.0，主版本号是 2，次版本号是 0。



## HIDL 基础语法

​	HIDL 的语法和 C 语言有点类似，支持嵌套声明，但不支持前向声明和预处理指令。以下是一些常用标记符和数据类型

### 标记符

| /* */                                        | 多行注释                                                     |
| -------------------------------------------- | ------------------------------------------------------------ |
| //                                           | 单行注释                                                     |
| [empty]                                      | 表面当前项的值为空                                           |
| ？                                           | 放置在项前，表明该项为可选项                                 |
| ...                                          | 表明该序列包含0个或多个如前述使用的分隔符隔开的项            |
| @entry                                       | 当前HAL模块被使用时应当被最先调用的接口                      |
| @exit                                        | 当前HAL模块被调用时应当被最后调用的接口                      |
| @callflow(next={"name_a","name_b","name_c"}) | 当前接口被调用后可能被调用的接口列表。其中name_a接口被调用的概率最大，name_c接口被调用的概率最小。如果只存在1个可能被调用的接口，那么花括号{ }可以省略不写。如果给定的接口名无效，则会导致VTS编译失败。 |
| @callflow(next={"*"})                        | 当前接口被调用后可能会调用任意接口                           |



### 数据类型

| **struct**                                       | 这个关键字定义一个结构体，格式与C++同                        |
| ------------------------------------------------ | ------------------------------------------------------------ |
| **union**                                        | 这个关键字定义一个联合体，格式与C++同                        |
| **MQDescriptorSync**<br />**MQDescriptorUnsync** | 这2个关键字分别定义同步和非同步的FMQ(Fast Message Queue)描述符 |
| **memory**                                       | 这个关键字用来声明HIDL中未被映射的共享内存                   |
| **pointer**                                      | 用这个关键字声明的pointer类型数据只能在HIDL内部使用          |
| **bitfield<T>模板**                              | 这个关键字用来定义一个与模板T相同的可进行位操作的数据。其中T是一个由用户定义的枚举数据类型 |
| **有限数组**                                     | 任何HIDL结构体中可被包含的数据类型都可以声明有限数组         |
| **字符串**                                       | 字符串在HIDL中以UTF8编码存储，所以在和由Java实现的接口进行交互时需要将编码格式转换为UTF16 |
| **vec<T>模板**                                   | 这个关键字用来定义一个包含模板T的可变大小的buffer数据。其中T可以是除句柄外的任何HIDL内建或用户自定义数据类型 |
| **用户自定义数据类型**                           | 用户可以自定义enum、struct、union类型的数据。定义enum数据的格式与C++11同，定义struct数据的格式与C同，定义union数据的格式与C同 |



### 关键字

| **interface** | 用于声明HAL模块中的一个接口，是构成.hal文件的基本单元，可以从其它interface继承而来 |
| ------------- | ------------------------------------------------------------ |
| **package**   | 用于声明当前.hal文件中各interface接口所属的包                |
| **import**    | 用于导入其它包里声明的interface或数据类型，以便在当前.hal文件中使用 |

