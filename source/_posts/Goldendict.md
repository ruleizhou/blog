---
title: Goldendict
date: 2019-08-13 15:27:29
categories:
- tools
tags:
- tools
---

​	GoldenDict 是一款不错的、与StarDict（星际译王）类似的词典软件。它使用 WebKit作为渲染核心，格式化、颜色、图像、链接等支持一应俱全；支持多种词典文件格式。并且可跨平台使用。加入google在线翻译功能更加完美。

# Linux

## 安装

```shell
sudo apt-get install goldendict
```

## 配置

### 导入词典

*   [词典下载](http://download.huzheng.org/zh_CN/)

*   选择合适的词典，下载并导入

    ```shell
    编辑 --> 词典 --> 文件 --> 添加词典目录
    ```

### 加入google翻译

*   下载[Translator shell](https://github.com/soimort/translate-shell)

*   编译安装

    ```shell
    make 
    sudo make install
    ```

*   配置

    ```shell
    编译-->词典-->程序-->添加
    类型: 纯文本
    名称: google
    命令: trans -e google -s en -t zh-CN -show-original y -show-original-phonetics n -show-translation y -no-ansi -show-translation-phonetics n -show-prompt-message n -show-languages y -show-original-dictionary n -show-dictionary n -show-alternatives n "%GDWORD%"
    ```

# Window

## 安装

[下载并安装](https://sourceforge.net/projects/goldendict/)

## 配置

### 导入词典

-   [词典下载](http://download.huzheng.org/zh_CN/)

-   选择合适的词典，下载并导入

    ```shell
    编辑 --> 词典 --> 文件 --> 添加词典目录
    ```

### 加入google翻译

*   下载[google-translate-for-goldendict](https://github.com/xinebf/google-translate-for-goldendict)

*   安装

    ```shell
    # 需要安装python3
    pip3 install requests
    ```

*   配置

    ```shell
    GoldenDict - 编辑 - 字典 - 字典来源 - 程式
    类型: 纯文字
    名称: Google Translate
    命令行: python H:\PathTo\googletranslate.py zh-CN %GDWORD%
    图示: H:\PathTo\google_translate.png
    ```

*   注意

    默认设置不能使用的可以尝试将 `http_host` 设为: `translate.google.cn`