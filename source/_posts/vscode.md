---
title: vscode
date: 2019-08-15 12:08:43
categories:
- tools
tags:
- tools
typora-root-url: vscode
---

# 安装

* [下载地址](<https://code.visualstudio.com/Download>)

* 安装

  ```shell
  sudo dpkg -i <file>.deb
  sudo apt-get install -f # Install dependencies
  ```

# 配置

## Go IDE

### 环境配置

* 配置GOPATH GOROOT

  ```shell
  export GOPATH=xxxx
  export GOROOT=xxxx
  ```

### 安装Go扩展

* vscode中搜索"go", 选择microsoft官方版本即可

### 安装相关工具包

通过Vscode安装:

* Ctrl + Shift + P
* 输入Go: install/update tools
* All Select,确定
* 部分安装失败，需要手动安装

手动安装

* 切换到GOPATH目录

* mkdir src/golang.org/x -p

* cd src/golang.org/x

* git clone <https://github.com/golang/tools.git> tools

* git clone <https://github.com/golang/lint.git>

* 打开vsCode终端，切换到 终端，进入“%GOPATH”目录,执行

  ```shell
  go install github.com/ramya-rao-a/go-outline
  go install github.com/acroca/go-symbols
  go install golang.org/x/tools/cmd/guru
  go install golang.org/x/tools/cmd/gorename
  go install github.com/josharian/impl
  go install github.com/rogpeppe/godef
  go install github.com/sqs/goreturns
  go install github.com/cweill/gotests/gotests
  go install github.com/ramya-rao-a/go-outline
  go install github.com/acroca/go-symbols
  go install golang.org/x/tools/cmd/guru
  go install golang.org/x/tools/cmd/gorename
  go install github.com/josharian/impl
  go install github.com/rogpeppe/godef
  go install github.com/sqs/goreturns
  go install github.com/cweill/gotests/gotests
  go install golang.org/x/lint/golintgo install go go go install golang.org/x/lint/golint
  ```

## C/C++ IDE

### 插件

* C/C++

  C/C++ 集成环境

* Code Runner

  设置如下：

  ```json
  code-runner.executorMap //的内容复制到右侧
  code-runner.runInTerminal: true
  "code-runner.clearPreviousOutput": true, //清除之前的输出
  "code-runner.saveFileBeforeRun": true //运行前保存文件
  ```

* Include Autocomplete

  头文件自动补全

* Path Autocomplete

  路径自动补全

* vscode-icons

  文件图标

* koroFileHeader

  [文件头部添加注释](https://www.imooc.com/article/29921)

* Bracket Pair Colorizer

  括弧着色器

* One Dark Pro

  主题插件

  设置

  ```shell
  ~/.vscode/extensions/zhuangtongfa.material-theme-2.18.1/themes
  add "editor.foreground":"#CD919E"
  ```