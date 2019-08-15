---
title: Selinux
date: 2019-08-15 14:55:01
categories:
- Android
tags:
- Android
- Selinux
typora-root-url: Selinux
---

# Selinux 概述

## 概述

- 两种安全机制

  - DAC   --> Discretionary Access Control

    在DAC里，如果一个应用获取了一个用户权限，如root，那么他的所有的操作都是基于这个用户的权限

  - MAC  --> Mandatory Access Control

    无论你是谁，甚至是有Root用户权限，文件权限为777，但每个动作都是需要被允许之后可以被执行。这里可以是在安全策略文件中被允许，与可以是用户手动允许。

- MAC基于LSM(Linux Security Module)标准实现的。

- SELinux全称Security-Enhanced Linux。SELinux在MAC上实现的，所以SELinux也是基于LSM标准。在Linux kernel 2.6后正式直接整合进Linux里面

- MAC的安全策略文件的作用就是表明了允许干什么。学名是TEAC（Type Enforcement Access Control）。简称TE。里面的语言被称为强制类型语言。

- Security Context，安全上下文。Security Context的作用就是相当于这些文件和进程的“身份证”。

- SELinux Mode

  - Permissve Mode（宽容模式）

    只会打印acvl log，不会进行拦截

  - Enforcing Mode（强制模式）

    真正的拦截并打印avc log

  - 模式查看

    - getenforce

    - cat /sys/fs/selinux/enforce

      1	--> Enforcing Mode

      0	--> Permissve Mode

    

## 架构

![](1.2-1.png)



- TE文件的规则和用户主动允许的动作等等会存储在图中的database里
- 当一个进程执行read动作（Event）被Security Server检测到的时候，DAC会作初步检查。然后把动作传进LSM，同时LSM会找到该进程和文件的安全上下文。
- SELinux在初始化的时候会实现一些由LSM提供的抽象函数（abstract）和把一些LSM回调（Hook）注册进LSM。LSM会读取SELinux里database的TE规则，或者在AVC（AccessVector cache）里寻找相应的规则。AVC相当于一个规则的缓存，加快读取的速度。找到相应的规则后又把它传回LSM，在LSM里做出最后的判断

![](1.2-2.png)

​	从图中可以看到，SEAndroid安全机制包含有内核空间（Kernel Space）和用户空间（User Space）两部分支持，以SELinux文件系统（SELinux File system）接口为边界。在内核空间，主要涉及到一个SELinux LSM模块。而在用户空间中，涉到安全上下文（Security Context）、安全服务（Security Server）和安全策略（SEAndroid Policy）等模块。这些内核空间模块和用户空间模块的作用以及交互如下：

- **内核空间的SELinux LSM模块**负责内核资源的安全访问控制。

- **用户空间的Security Context**描述的是资源安全上下文。

  从上文我们知道，SEAndroid的安全访问策略就是在资源（进程、文件等）的安全上下文基础上实现的

- **用户空间的SEAndroid Policy**描述的是资源安全访问策略。

  系统在启动的时候，用户空间的Security Server会将这些安全访问策略加载内核空间的SELinux LSM模块中去。这是通过SELinux文件系统接口实现的。

- **用户空间的Security Server**一方面需要到用户空间的Security Context去检索对象的安全上下文，另一方面也需要到内核空间去操作对象的安全上下文。

- **用户空间的libselinux库**封装了对SELinux文件系统接口的读写操作。

  用户空间的Security Server访问内核空间的SELinux LSM模块时，都是间接地通过libselinux进行的。这样可以将对SELinux文件系统接口的读写操作封装成更有意义的函数调用。

  用户空间的Security Server到用户空间的Security Context去检索对象的安全上下文时，同样也是通过libselinux库来进行的。



## 流程

![](1.2-3.png)

![](1.2-4.png)

- 从上层到驱动层的调用流程，但是我们重点关注sContext

- file_contexts

  系统中所有file_contexts安全上下文

- seapp_contexts

  app安全上下文

- property_contexts

  属性的安全上下文

- service_contexts

  service文件安全上下文

- genfs_contexts

  虚拟文件系统安全上下文



# 基础概念

## 主体(Subject)/客体(Object)

​	SELinux两个最基本的对象是主体（Subject）和客体（Object）。主体和客体分别对应的是“进程”和“文件”。这里的文件并不单指的是实际存在的文件，而是指Linux里“一切皆文件”里指的文件。如Socket，系统属性等

​	在SEAndroid，对主体和客体进行了进一步形式上的封装和扩展，其实差不多。SEAndroid里细分为：系统文件，服务，系统属性，Bindert和Socket。这里的系统属性指的是build.prop里的属性，也是getprop命令查询出来的属性。



## 安全上下文

​	全策略又是建立在对象的安全上下文的基础上的，安全上下文实际上就是一个附加在对象上的标签（label）。这个标签实际上就是一个字符串，它由四部分内容组成，分别是**SELinux用户**、**SELinux 角色**、**类型**、**安全级别**，每一个部分都通过一个冒号来分隔，格式为“user:role:type:rank”。

- 文件的安全上下文

  - ls -Z init.rc

    *-rwxr-x--- root     root    **u:object_r:rootfs:s0** init.rc*

- 进程的安全上下文

  - ps -Z

    LABEL                          USER     PID  PPID  NAME

    **u:r:init:s0**                    root         1       0        /init*





### 用户和角色

​	对于进程来，SELinux用户和SELinux角色只是用来限制进程可以标注的类型。对于文件来说，SELinux用户和SELinux角色就可以完全忽略不计。

​	**进程**的安全上下文中，用户固定为 u，角色固定为 r。

​	**文件**的安全上下文中，用户固定为 u，角色固定为 obejct_r

### 安全级别

​	在SELinux中，安全级别是可选的，也就是说，可以选择启用或者不启用。通常在进程及文件的安全上下文中安全级别都设置为s0

### 类型

​	**进程**的安全上下文的类型称为domain，**文件**的安全上下文中的类型称为file_type。

​	上面我们看到进程init的安全上下文为u:r:init:s0，type为init，这也是合法的，为什么呢？在Android中我们可以通过如下语句定义type:

*type init domain;*

即将domain设置为init的属性，这样就可以用init为type来描述进程的安全上下文了。



## 安全上下文描述

​	安全上下文根据主体对象不同，有如下几种，在Android O上一般都存放在/system/sepolicy/private/目录下。

### mac_permissions.xml

​	用于给不同签名的App分配不同的seinfo字符串，例如，在AOSP源码环境下编译并且使用平台签名的App获得的seinfo为“platform”。这个seinfo描述的是其实并不是安全上下文中的Type，它是用来在另外一个文件seapp_contexts中查找对应的Type的。

![](1.2-5.png)



### Seapp_contexts

​	用于声明APP进程和创建数据目录的安全上下文，O上将该文件拆分为plat和nonplat 前缀的两个文件，plat前缀的文件用于声明system app，nonplat前缀的文件用于声明vendor app。

![](1.2-6.png)

​	从前面的分析可知，对于使用平台签名的App来说，它的seinfo为“platform”。这样我们就可以知道，使用平台签名的**App**所运行在的进程domain为“platform_app”，并且它的数据文件的file_type为“platform_app_data_file”。



### File_contexts

​	用于声明文件的安全上下文，plat前缀的文件用于声明system、rootfs、data等与设备无关的文件。Nonplat 用于声明vendor、data/vendor 等文件。

![](1.2-7.png)

​	如上，/system目录包括子目录和文件的安全上下文为u:object_r:system_file:s0，这意味着只有有权限访问Type为system_file的资源的进程才可以访问这些文件。

### Service_contexts

​	用于声明java service 的安全上下文， O上将该文件拆分为plat和nonplat 前缀的两个文件，但nonplat前缀的文件并没有具体的内容（vendor和system java service不允许binder 操作）

![](1.2-8.png)

​	如上，service为netd的安全上下文为u:object_r:netd_service:s0，这意味着只有有权限访问Type为netd_service的资源的进程才可以访问这些service。

### Property_contexts

​	用于声明属性的安全上下文，plat 前缀的文件用于声明system属性，nonplat前缀的文件用于声明vendor 属性。

![](1.2-9.png)

​	如上，ril.开头的属性的安全上下文为u:object_r:radio_prop:s0，这意味着只有有权限访问Type为radio_prop的资源的进程才可以访问这些属性。

### Hwservice_contexts

​	O 上新增文件，用于声明HIDL service 安全上下文。

![](1.2-10.png)

​	如上，android.frameworks.sensorservice::ISensorManager的hw service的安全上下文为u:object_r:fwk_sensor_hwservice:s0，这意味着只有有权限访问Type为fwk_sensor_hwservice的资源的进程才可以访问这些hw service。



# 策略文件

## 文件预览

```shell
sepolicy/attributes		
	-->所定义的attributes都在这个文件
sepolicy/access_vectors	
	-->对应每一个class可以别允许执行的命令
sepolicy/roles			
	-->Android中只定义了一个role，名字就是r，将r和attribute domain关联起来
sepolicy/users			
	-->其实是将user与roles进行了关联，设置了user的安全级别，s0为最低级是默认的级别
sepolicy/security_class	
	-->指的是上文命令中的class，个人认为这个class的内容是指在android运行过程中，程序或者系统可能用到的操作的模块 
sepolicy/te_macros		
	-->系统定义的宏全在te_macros文件 
sepolicy/*.te			
	-->一些策略的文件，包含了各种运行的规则
```



## 基本定义

### 定义类型

```shell
#定义type
#格式：type 类型名称 [alias 别名集] [,属性集]
type XXXXX;
tyep  alias{ aa , bb , cc }     XX ,  YY;
```

在TE中所有的东西都会被抽象成类型。进程抽象成类型、资源抽象成类型。属性是类型的集合。所以在TE规则中的最小单位就是类型



### 申明别名

```shell
#使用typealias申明别名
type mozilla_t, domain;
typealias mozilla_t alias netscape_t;
#上面两句等于下面一句
type mozilla_t alias netscape_t, domain;
```



### 声明属性

```shell
#格式:attribute 属性名
attribute domain；
```



### 类型与属性关联

类型与属性的关联有两种方式，一种是定义类型的时候进行关联

```shell
#格式：type 类型名称 [alias 别名集] [,属性集]
type app_data_file, file_type, data_file_type;
```

另一种是使用typeattribute进行类型与属性的关联

```shell
#格式：typeattribute 类型名 [属性集]
type httpd_user_content_t; 
typeattribute httpd_user_content_t file_type, httpdcontent;
```



## 访问向量(AV)

访问向量(AV)规则用来描述主体对客体的访问许可。通常有类AV规则：

- allow

  表示允许主体对客体执行许可的操作。

- neverallow

  表示不允许主体对客体执行制定的操作。

- auditallow

  表示允许操作并记录访问决策信息。

- dontaudit

  表示不记录违反规则的决策信息，切违反规则不影响运行。

  

## 语法

格式：

```shell
rule_name source_type target_type:class perm_s
```

  rule_name 就是上面说的allow 等

注意：

​	如果有多个source_type，target_type，class或perm_set，可以用”{}”括起来；

* `”~”`号，表示除了”~”以外；
* `”-”`号，表示去除某项内容；
* `“*”`号，表示所有内容



### Demo1

```shell
allow appdomain zygote_tmpfs:file read;
```

允许appdomain域的进程对zygote_tmpfs类型的文件（file）执行read操作；



### Demo2

```shell
allow zygote appdomain:process { getpgid setpgid };
```

允许zygote域的进程对appdomain类型的进程执行getpgid和setpgid操作；



### Demo3

```shell
allow bluetooth { tun_device uhid_device hci_attach_dev }:chr_file { read write }
```

允许bluetooth域的进程对tun_device, uhid_device, hci_attach_dev类型的字符文件（chr_file）执行读写操作；



### Demo4

```shell
allow init {

  shell_data_file

  app_data_file

}:{ chr_file file } ~{entrypoint execute_no_trans execmod execute relabelto};
```

表示允许init域的进程对shell_data_file,app_data_file类型的字符文件（chr_file），普通文件（file）执行除了entrypointexecute_no_trans execmod execute relabelto以外的操作；



### Demo5

```shell
allow init { file_type -system_file }:dir relabelto;
```

”-”符号表示去除某项，即允许init域的进程对file_type类型中除了system_file类型外的目录执行relabelto操作



### Demo6

```shell
allow system_server self:netlink_selinux_socket *;
```

”*”符号表示所有内容，即system_server域的进程能够对system_server类型执行所有netlink_selinux_socket类相关的操作；



### Demo7

```shell
neverallow {

  domain -keystore

} keystore_data_file:dir ~{ open create read getattr setattr search relabelto };
```

 简单来说就是不允许除keystore域外的其它域对keystore_data_file类型的目录执行open，create等操作；

特别注意，前面提到权限必须显示声明，没有声明的话默认就没有权限。那么neverallow语句就没有存在的必要了，因为“无权限”是不需要声明的。确实如此，neverallow语句的作用只是在生成安全策略文件时进行检查，判断是否有违反neverallow语句的策略存在。



# Type转换

## 概述

type类型转换分两种情况，一种是进程主体type转换，另一种是资源客体type转换。

- DT -->Domain Transition

  某个进行的Domain切换到一个合适的Domain中去

- TT -->Type  Transition

  假设目录A的SContext为u:r:dir_a，那么默认情况下，在该目录下创建的文件的SContext就是u:r:dir_a，如果想让它的SContext发生变化，那么就需要TypeTransition



## DT转换

语法格式：

```shell
type_transition  source_type  target_type : class  default_type
```

表示source_type域的进程在对target_type类型的资源进行object class定义的许可操作时，进程会切换到default_type域中.

例如：

```shell
type_transition init shell_exec:process init_shell
```

当init域的进程执行（process）shell_exec类型的可执行文件时，进程会从init域切换到init_shell域.

但是上述策略要想被执行的话,至少还有三个前提:

- 首先，你得让init域的进程要能够执行shell_exec类型的文件

  ```shell
  allow init shell_exec:file execute;
  ```

- 告诉SELinux，允许init域的进程切换到init_shell域

  ```shell
  allow init init_shell:process transition;
  ```

- 告诉SELinux，域切换的入口（entrypoint）是执行shell_exec类型的文件

  ```shell
  allow init_shell shell_exec:file entrypoint;
  ```

这样看来转换需要的步骤还挺多的,好在Android Selinux定义了一个宏,可以一次性做完上面的四步操作.

```shell
domain_auto_trans(init, shell_exec, init_shell)
```



## TT转换

```shel
file_type_trans(domain, dir_type, file_type)
```

该宏的意思就是当domain域的进程在dir_type类型的目录创建文件时，该文件的SContext应该是file_type类型

