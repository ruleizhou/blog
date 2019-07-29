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

## 命令规范

### 目录结构和文件命令

### 软件包名称

### 版本

### 导入

### 接口名称

### 函数

### 结构体/联合字段名称

### 类型名称

### 枚举值

## 备注

### 文件备注

### TODO 备注

### 接口/函数备注

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

        