---
title: Linux下查找和安装依赖的.so文件
typora-root-url: linux install so
date: 2019-11-06 13:53:21
categories:
- Others
tags:
- Linux
---

介绍Linux中出现的缺少库的解决方法。

# 查看程序所依赖的库.so文件

ldd的作用是打印可执行档依赖的共享库文件。它是glibc的一部分，由Roland McGrath和Ulrich Drepper维护

但是ldd本身不是一个程序，而仅是一个shell脚本：

```shell
$ which ldd
/usr/bin/ldd
$ file /usr/bin/ldd 
/usr/bin/ldd: Bourne-Again shell script text executable
```

使用方法如下：

```shell
$ ldd $HOME/.webex/1324/*.so | grep 'not found'
    libgtk-x11-2.0.so.0 => not found
    libgdk-x11-2.0.so.0 => not found
    libXmu.so.6 => not found
    libXtst.so.6 => not found
    libjawt.so => not found
    libjawt.so => not found
    libXmu.so.6 => not found
    libpangoxft-1.0.so.0 => not found
    libXft.so.2 => not found
    libpangoft2-1.0.so.0 => not found
    libpangox-1.0.so.0 => not found

```



# 安装.so查找工具

```shell
sudo apt-get install apt-file
apt-file update
```



# 查找.so所在的deb包

```shell
$ apt-file search libXmu.so.6
libxmu6: /usr/lib/x86_64-linux-gnu/libXmu.so.6
libxmu6: /usr/lib/x86_64-linux-gnu/libXmu.so.6.2.0
libxmu6-dbg: /usr/lib/debug/usr/lib/x86_64-linux-gnu/libXmu.so.6.2.0
```



# 安装对应的deb包

```shell
sudo apt-get install -y libxmu6
```

