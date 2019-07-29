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



# HIDL 编程规范

[参考网站](https://blog.csdn.net/shift_wwx/article/details/86525761)

​	HIDL 代码样式类似于 Android 框架中的 C++ 代码，缩进 4 个空格，并且采用混用大小写的文件名。软件包声明、导入和文档字符串与 Java 中的类似，只有些微差别。

下面针对 `IFoo.hal` 和 `types.hal` 的示例展示了 HIDL 代码样式。

`hardware/interfaces/foo/1.0/IFoo.hal`

```
/*
 * (License Notice)
 */
 
package android.hardware.foo@1.0;
 
import android.hardware.bar@1.0::IBar;
 
import IBaz;
import IFooClientCallback;
 
/**
 * IFoo is an interface that…
 */
interface IFoo {
 
    /**
     * This is a multiline docstring.
     * @return result 0 if successful, nonzero otherwise.
     */
     foo() generates (FooStatus result);
 
    /**
     * Restart controller by power cycle.
     * @param bar callback interface that…
     * @return result 0 if successful, nonzero otherwise.
     */
    powerCycle(IBar bar) generates (FooStatus result);
 
    /** Single line docstring. */
    baz();
 
    /**
     * The bar function.
     * @param clientCallback callback after function is called
     * @param baz related baz object
     * @param data input data blob
     */
    bar(IFooClientCallback clientCallback,
        IBaz baz,
        FooData data);
 
}
```

`hardware/interfaces/foo/1.0/types.hal`

```hal
/*
 * (License Notice)
 */
 
package android.hardware.foo@1.0;
 
/** Replied status. */
enum Status : int32_t {
    OK,
    ERR_ARG, // invalid arguments
    ERR_UNKNOWN = -1, // note, no transport related errors
};
 
struct ArgData {
    int32_t[20]  someArray;
    vec<uint8_t> data;
};
```

## 命令规范

### 目录结构和文件命令

目录结构应如下所示：

*   Root-dir
    *   Module
        *   SubModule
            *   Version
                *   Android.bp
                *   IInterface_1.hal
                *   IInterface_2.hal
                *   ...
                *   IInterface_n.hal
                *   types.hal (可选)

其中：

*   Root-dir

    *   hardware/interfaces 核心HIDL软件包
    *   vendor/VENDOR/interfaces 供应商软件包

*   Module

    描述系统的小写字词。如果多个字词，需使用嵌套式SubModule

*   Version

    版本号，格式：major.minor

*   IInterface_x

    接口名称

### 软件包名称

软件包名称必须采用[完全限定名称(FQN)](https://source.android.com/devices/architecture/hidl/code-style#fqn)格式（称为：PACKAGE-NAME）。格式如下：

```shell
PACKAGE.MODULE[.SUBMODULE[.SUBMODULE[…]]]@VERSION
```

### 版本

格式如下：

```shell
MAJOR.MINOR
```

MAJOR 和 MINOR 版本都应该是一个整数。

### 导入

格式如下：

*   完整软件包导入

    `import PACKAGE-NAME`

*   部分导入

    `import PACKAGE-NAME::UDT;`（或者，如果导入的类型是在同一个软件包中，则为 `import UDT;`）。

*   仅类型导入

    `import PACKAGE-NAME::types;`

如果当前软件包types.hal存在，则自动导入。

在软件包声明之后(在导入之前)，添加一个空行。每个导入都应占用一行，且不能缩进。按一下顺序对导入进行分组：

*   android.harder 软件包（使用完全限定名称）
*   vendor.VENDOR软件包（使用完全限定名称）
    *   每个供应商应为一组
    *   按字母顺序对供应商进行排序
*   源自同一个软件包的其他接口导入（使用简单名称）

在组与组之间添加一行空行。每个组内，按照字母顺序对导入进行排序。

### 接口名称

接口名称必须以 `I` 开头，后跟 `UpperCamelCase`/`PascalCase` 名称。名称为 `IFoo` 的接口必须在文件 `IFoo.hal` 中定义。此文件只能包含 `IFoo` 接口的定义（接口 `INAME` 应位于 `INAME.hal` 中）。

### 函数

对于函数名称、参数和返回变量名称，请使用 `lowerCamelCase`。例如：

```C++
open(INfcClientCallback clientCallback) generates (int32_t retVal);
oneway pingAlive(IFooCallback cb);
```

### 结构体/联合字段名称

对于结构体/联合字段名称，请使用 `lowerCamelCase`。例如：

```C++
    struct FooReply {
        vec<uint8_t> replyData;
    }
```

### 类型名称

类型名称指结构体/联合定义、枚举类型定义和 `typedef`。对于这些名称，请使用 `UpperCamelCase`/`PascalCase`。例如：

```C++
enum NfcStatus : int32_t {
    /*...*/
};
struct NfcData {
    /*...*/
};
```

### 枚举值

枚举值应为 `UPPER_CASE_WITH_UNDERSCORES`。将枚举值作为函数参数传递以及作为函数返回项返回时，请使用实际枚举类型（而不是基础整数类型）。例如：

```C++
enum NfcStatus : int32_t {
    HAL_NFC_STATUS_OK               = 0,
    HAL_NFC_STATUS_FAILED           = 1,
    HAL_NFC_STATUS_ERR_TRANSPORT    = 2,
    HAL_NFC_STATUS_ERR_CMD_TIMEOUT  = 3,
    HAL_NFC_STATUS_REFUSED          = 4
};
```

注意：

​	枚举类型的基础类型是在冒号后显式声明的。因为它不依赖于编译器，所以使用实际枚举类型会更明晰

## 备注

### 文件备注

​	每个文件的开头都应为相应的许可通知。对于核心 HAL，该通知应为 [development/docs/copyright-templates/c.txt](https://android.googlesource.com/platform/development/+/master/docs/copyright-templates/c.txt) 中的 AOSP Apache 许可。请务必更新年份，并使用 `/* */` 样式的多行备注（如上所述）。

​	您可以视需要在许可通知后空一行，后跟变更日志/版本编号信息。使用 /* */ 样式的多行备注（如上所述），在变更日志后空一行，后跟软件包声明。

### TODO 备注

TODO 备注应包含全部大写的字符串 `TODO`，后跟一个冒号。例如：

```C++
// TODO: remove this code before foo is checked in.
```

只有在开发期间才允许使用 TODO 备注；TODO 备注不得存在于已发布的接口中。

### 接口/函数备注

对于多行和单行文档字符串，请使用 `/** */`。对于文档字符串，请勿使用 `//`。

接口的文档字符串应描述接口的一般机制、设计原理、目的等。函数的文档字符串应针对特定函数（软件包级文档位于软件包目录下的 README 文件中）。

```hal
/**
 * IFooController is the controller for foos.
 */
interface IFooController {
    /**
     * Opens the controller.
     * @return status HAL_FOO_OK if successful.
     */
    open() generates (FooStatus status);
 
    /** Close the controller. */
    close();
};
```

必须为每个参数/返回值添加 `@param` 和 `@return`：

*   必须为每个参数添加 `@param`。其后应跟参数的名称，然后是文档字符串
*   必须为每个返回值添加 `@return`。其后应跟返回值的名称，然后是文档字符串

```hal
/**
 * Explain what foo does.
 * @param arg1 explain what arg1 is
 * @param arg2 explain what arg2 is
 * @return ret1 explain what ret1 is
 * @return ret2 explain what ret2 is
 */
foo(T arg1, T arg2) generates (S ret1, S ret2);
```

## 格式

### 软件包声明

### 函数声明

### 注释

### 枚举声明

### 结构体声明

### 数组声明

### 矢量声明





# hidl-gen 使用

系统定义的所有的`.hal`接口，都是通过`hidl-gen`工具转换成对应的代码。`hidl-gen`源码路径：system/tools/hidl，是在ubuntu上可执行的二进制文件

## 编译工具

```shell
make -j18 hidl-gen
```

编译之后会在out 下生成，详细看out/host/linux-x86/bin/hidl-gen。

## 使用方法

```shell
hidl-gen -o output_path -L language (-r interface:root) hidl_name
```

*   -L

    语言类型，包括c++, c++-headers, c++-sources, export-header, c++-impl, java, java-constants, vts, makefile, androidbp, androidbp-impl, hash等。hidl-gen可根据传入的语言类型产生不同的文件。

*   hidl_name

    完全限定名称的输入文件。格式：`package@version`

*   -r

    格式：package:path，可选，对hidl_name对应的文件来说，用来指定包名和文件所在的目录到Android系统源码根目录的路径。如果没有制定，前缀默认是：android.hardware，目录是Android源码的根目录

*   -o

    存放hidl-gen产生的中间文件的路径

## 实例

1.  在`hardware/interfaces/`目录下新建`han/1.0/`目录，并创建接口文件`IHan.hal`，目录结构如下：

    ```shell
    └── 1.0
        └── IHan.hal
    ```

    在IHal.hal文件中只有一个接口IHan和一个方法helloWorld(string name)，代码如下：

    ```hal
    package android.hardware.han@2.0;
     
    interface IHan{
                helloWorld(string name) generates (string result);
    };
    ```

2.  自动生成对应的C++模板文件

    ```shell
    PACKAGE=android.hardware.han@1.0
    LOC=hardware/interfaces/han/1.0/default/
    hidl-gen -o $LOC -Lc++-impl -r android.hardware:hardware/interfaces -r  android.hidl:system/libhidl/transport $PACKAGE
    ```

    自动生成模板后目录结构如下：

    ```shell
    └── 1.0
        ├── default
        │   ├── Han.cpp
        │   └── Han.h
        └── IHan.hal
    ```

    C++ 代码如下：

    ```c++
    #include "Han.h"
    
    namespace android {
    namespace hardware {
    namespace han {
    namespace V1_0 {
    namespace implementation {
    
    // Methods from ::android::hardware::han::V1_0::IHan follow.
    Return<void> Han::helloWorld(const hidl_string& name, helloWorld_cb _hidl_cb) {
        // TODO implement
        return Void();
    }
    
    // Methods from ::android::hidl::base::V1_0::IBase follow.
    
    //IHan* HIDL_FETCH_IHan(const char* /* name */) {
        //return new Han();
    //}
    //
    }  // namespace implementation
    }  // namespace V1_0
    }  // namespace han
    }  // namespace hardware
    }  // namespace android
    ```

    如果是直通模式HAL，需要开启HIDL_FETCH_IHan。

3.  自动生成C++模板对应的Android.bp文件

    ```shell
    hidl-gen -o $LOC -Landroidbp-impl -r android.hardware:hardware/interfaces  -r android.hidl:system/libhidl/transport $PACKAGE
    ```

    执行命令后目录结构如下：

    ```shell
    └── 1.0
        ├── default
        │   ├── Android.bp
        │   ├── Han.cpp
        │   └── Han.h
        └── IHan.hal
    ```

    Android.bp内容如下：

    ```makefile
    cc_library_shared {
        // FIXME: this should only be -impl for a passthrough hal.
        // In most cases, to convert this to a binderized implementation, you should:
        // - change '-impl' to '-service' here and make it a cc_binary instead of a
        //   cc_library_shared.
        // - add a *.rc file for this module.
        // - delete HIDL_FETCH_I* functions.
        // - call configureRpcThreadpool and registerAsService on the instance.
        // You may also want to append '-impl/-service' with a specific identifier like
        // '-vendor' or '-<hardware identifier>' etc to distinguish it. 
        name: "android.hardware.han@1.0-impl",
        relative_install_path: "hw",
        // FIXME: this should be 'vendor: true' for modules that will eventually be
        // on AOSP.
        proprietary: true,
        srcs: [
            "Han.cpp",
        ],  
        shared_libs: [
            "libhidlbase",
            "libhidltransport",
            "libutils",
            "android.hardware.han@1.0",
        ],  
    }
    ```

    默认适合直通模式HAL，如果是绑定模式需要按照提示进行修改。

4.  使用脚本`update-makefiles.sh`，来更新Makefile。自动在`hardware/interfaces/han/1.0`目录下生成Android.bp

    ```shell
    └── 1.0
        ├── Android.bp
        ├── default
        │   ├── Android.bp
        │   ├── Han.cpp
        │   └── Han.h
        └── IHan.hal
    ```

5.  在`hardware/interfaces/han/1.0/default`目录下新建`service.cpp` `android.hardware.han@1.0-service.rc`。

    1.  `android.hardware.han@1.0-service.rc` 实现

        ```shell
        service han_hal_service /vendor/bin/hw/android.hardware.han@1.0-service
            class   hal 
            user    system
            group   system
        ```

    2.  `service.cpp` 实现

        ```C++
        #define LOG_TAG "android.hardware.han@1.0-service"
        #include <android/hardware/han/1.0/IHan.h>
        #include <hidl/LegacySupport.h>
        using android::hardware::han::V1_0::IHan;
        using android::hardware::defaultPassthroughServiceImplementation;
        
        int main(){
            return defaultPassthroughServiceImplementation<IHan> (); 
        }
        ```

    3.  在`hardware/interfaces/han/1.0/default`的Android.bp中增加如下内容：

        ```makefile
        cc_binary {
            name: "android.hardware.han@1.0-service",
            init_rc: ["android.hardware.han@1.0-service.rc"],
            vendor: true,
            proprietary: true
            relative_install_path: "hw",
            srcs: [
                "service.cpp",
            ],  
        
            shared_libs: [
                "libcutils",
                "liblog",
                "libhidlbase",
                "libhidltransport",
                "libhardware",
                "libutils",
                "android.hardware.han@1.0",
            ],  
        
        }
        ```

        至此更HAL相关代码已经实现完成。

6.  编译生成服务端和客服端要用的各种库文件。

    ```shell
    ./hardware/interfaces/update-makefiles.sh
    mmm hardware/interfaces/han/1.0
    ```

    

7.  在`manifest.xml`文件里添加接口定义

    ```xml
    <hal format="hidl">
    	<name>android.hardware.han</name>
    	<transport>hwbinder</transport>
    	<version>1.0</version>
    	<interface>
    		<name>IHan</name>
    		<instance>default</instance>
    	</interface>
    </hal>
    ```

    

8.  使用C++实现客户端调用

    在`hardware/interfaces/han/1.0`目录下创建test目录。并在test目录下新建HanTest.cpp Android.bp

    1.  C++实现客户端调用

        ```C++
        #define LOG_TAG "android.hardware.han@1.0-service"
        #include <hidl/Status.h>
        #include <hidl/HidlSupport.h>
        #include <hidl/LegacySupport.h>
        #include <utils/misc/h>
        #include <stdio.h>
        
        using ::android::hardware::hidl_string;
        using ::android::sp;
        using android::hardware::han::V1_0::IHan;
        
        int main(){
            android::sp<IHan> service = IHan::getService();
            if(service == NULL){
                printf("Failed to get service\n");
                return -1;
            }
            service->helloWorld("Hello Woprld");
            return 0;
        }
        ```

    2.  Android.bp

        ```makefile
        cc_binary {
            name: "han_client",
            defaluts: ["hidl_defaults"],
            relative_install_path: "hw",
            proprietary: true,
            srcs: [
                "HanTest.cpp",
            ],  
            shared_libs: [
                "liblog",
                "libhardware",
                "libhidlbase",
                "libhidltransport",
                "libutils",
                "android.hardware.han@1.0",
            ],  
        }
        ```

        