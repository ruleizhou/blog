---
title: Kernel  Command Line
typora-root-url: Kernel-Command-Line
date: 2019-08-23 18:02:07
categories:
- Kernel
tags:
- Kernel
---

# 处理模型

* Linux kernel的启动包括很多组件的初始化和相关配置，这些配置参数一般是通过command line进行配置的。
* 要处理的对象是一个字符串，其中包含了各种配置信息，通常各个配置之间通过空格进行分离，每个配置的表达形式是如：param=value1,value2 或者很简单就是一个rw
* 那么kernel就需要提供对这些参数进行处理的处理函数列表。根据参数的作用以及执行期的先后不同，这些处理函数被定义到不同的段中。针对每一个参数，Kernel都会到相应的段中查找相应的处理函数，最终进行各个组件的配置。

# 格式配置

常见的配置格式如下：

```shell
console=ttySAC0,115200 root=nfs nfsroot=192.168.1.9:/source/rootfs initrd=0x10800000,0x14af47
```

# 配置方式

## bootloader动态配置

由bootloader进行参数配置，command line将做为atag_list的一个节点传递到Kernel。

## kernel静态配置

通过make menuconfig进行配置：运行后配置boot options->Default kernel command string。该配置将被静态编译到Kernel中，通过变量default_command_line访问。

# 解析配置

## 相关定义

```c
//include/asm-generic/vmlinux.lds.h 设置对齐方式
#define INIT_SETUP(initsetup_align)				\
		. = ALIGN(initsetup_align);				\
		VMLINUX_SYMBOL(__setup_start) = .;		\
		KEEP(*(.init.setup))					\
		VMLINUX_SYMBOL(__setup_end) = .;
```

.init.setup段内存储的是不是参数，而是command line参数所需要的处理函数。这些处理函数是通过_setup宏来定义的。对于宏__setup，可以在include/linux/init.h

```c
#define early_param(str, fn)						\
	__setup_param(str, fn, fn, 1)

#define __setup(str, fn)						\
	__setup_param(str, fn, fn, 0)

#define __setup_param(str, unique_id, fn, early)				\
	static const char __setup_str_##unique_id[] __initconst		\
		__aligned(1) = str; 									\
	static struct obs_kernel_param __setup_##unique_id			\
		__used __section(.init.setup)							\
		__attribute__((aligned((sizeof(long)))))				\
		= { __setup_str_##unique_id, fn, early }

```

可以看到还有一个宏 **early_param**，它与宏**__setup**的定义相似，只不过最后一个宏参数是1而不是0。1表示需要提前处理的参数。

## 解析

### 相关变量

* **boot_command_line**

  存在于.init.data段

### 主要函数

![](/1-1.png)

