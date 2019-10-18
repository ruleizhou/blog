---
title: deepin-wine
typora-root-url: deepin-wine
date: 2019-10-18 12:14:14
categories:
- Deepin
tags:
- Deepin
---

# 配置deepin-wine

## 配置直接运行exe的方法

* 新建`deepin-wine.desktop`

  ```shell
  touch deepin-wine.desktop
  ```

* 配置`deepin-wine.desktop`文件

  ```shell
  [Desktop Entry]
  Name=Deepin-wine
  Exec=deepin-wine %F
  Type=Application
  MimeType=text/plain
  ```

* 拷贝到`/usr/share/applications/`

  ```shell
  sudo mv deepin-wine.desktop /usr/share/applications/
  ```

  

# install win32程序

## 创建容器

* 容器就是win32程序运行的环境，可以理解为一个极小的windows，在Linux下面实际对应一个文件目录，如QQ对应的容器目录是~/.deepinwine/Deepin-QQ。
* 创建容器最简单实用的方法就是将deepin维护的容器拷贝一份，如将QQ的容器拷贝一份到用户目录。cp -r ~/.deepinwine/Deepin-QQ ~/.bottle
* 创建一个干净的容器可以用如下命令：WINEPREFIX=~/.bottle deepin-wine winecfg 。但是这样可能会有一些字体乱码的问题。

## 运行程序

* 只通过deepin-wine *.exe 可以运行程序，但是默认通~/.wine的容器运行，~/.wine是wine默认生成的干净的容器，没有适配应用运行可能会有一些问题，所以最好通过上一步创建好的容器，可以每一个应用对应一个容器，不同的应用可能会需要不同的配置。
* 通过WINEPREFIX的环境变量可以指定容器运行程序。如WINEPREFIX=~/.bottle deepin-wine *.exe