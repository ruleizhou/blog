---
title: htc
date: 2019-09-30 10:28:11
categories:
- Works
tags:
- HTC
typora-root-url: htc
password: han
abstract: Welcome to personal work note, enter password to read
message: HAN
---

# Install SsdTest_Tool

## 下载

[下载链接](http://jenkins.htc.com/SsdTest/)

根据项目选择合适的版本

## 安装

解压下载的软件包并执行脚步

## 配置

默认安装好后log功能即可使用，如果发现/data/htclog/下没有log文件输出，这需要进行额外配置。

- 重启到bootloader
- htc_fastboot oem writeconfig 5 1