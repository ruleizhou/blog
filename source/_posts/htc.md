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

# 1 HTC Tools

## 1.1 Install SsdTest_Tool

### 1.1.1 下载

[下载链接](http://jenkins.htc.com/SsdTest/)

根据项目选择合适的版本

### 1.1.2 安装

解压下载的软件包并执行脚步

### 1.1.3 配置

默认安装好后log功能即可使用，如果发现/data/htclog/下没有log文件输出，这需要进行额外配置。

- 重启到bootloader
- htc_fastboot oem writeconfig 5 1

## 1.2 htc fastboot

### 1.2.1 Project PID

![](/PID.png)

### 1.2.2Command

| 命令                                                        | 说明                                       |
| ----------------------------------------------------------- | ------------------------------------------ |
| htc_fastboot devices                                        | 列出所有已连接的设备                       |
| htc_fastboot oem battcheck $VAR                             | Download下检查电池状态                     |
| htc_fastboot oem writeconfig 6 2                            | 开启 uart kernel log                       |
| htc_fastboot oem writeconfig 8 20000                        | 开启USB diag                               |
| htc_fastboot oem write 8 8                                  | enable crashdump                           |
| htc_fastboot flash zip $ZIP_FILE                            | download zip file                          |
| htc_fastboot oem listpartition                              | list partition table                       |
| htc_fastboot oem listpartition 1                            | list partition table by HEX and UFS number |
| htc_fastboot dump emmc $OUTPUT_NAME $START_ADDR $SIZE       | dump storage from emmc                     |
| htc_fastboot dump ufs$NUMBER $OUTPUT_NAME $START_ADDR $SIZE | dump storage from ufs                      |
| htc_fastboot oem readmid/readpid                            | read pid/mid                               |
| htc_fastboot oem writepid/writemid  pid/mid                 | write pid/mid                              |

# 2 Log获取

## 2.1 XBL Log

- 关闭Ramdump mode

  ```shell
  adb reboot bootloader
  htc_fastboot oem writeconfig 8 0
  ```

  

- 触发system crash

  ```shell
  adb shell
  echo c > /proc/sysrq-trigger
  ```

- 重启设备

- 导出xbl log

  ```shell
  adb pull /sys/fs/pstore/console-ramoops  .
  ```

## 2.2 TZ Log

TZ log 保存路径

```shell
adb shell cat /d/tzdbg/qsee_log > qsee.log
adb shell cat /d/tzdbg/log > tzbsp.log
```



# 3 Debug

## 3.1 动态调试

pr_debug/dev_dbg 可实现动态调试. 可以通过conctrol获取哪些支持动态调试的文件.在需要的时候开启log的打印.

- 确认是否支持动态调试

  - 如果定义了CONFIG_DYNAMIC_DEBUG，就使用动态debug机制dynamic_pr_debug();
  - 如果定义了DEBUG，就使用printk(KERN_DEBUG...)
  - 默认情况下，不打印。

- 获取支持动态调试的文件

  ```shell
  cat /sys/kernel/debug/dynamic_debug/control
  ```

- 打开动态调试

  ```shell
  //打开某一行的log
  echo 'file filename line number +p' > /sys/kernel/debug/dynamic_debug/control
  ```

  ```shell
  //打开某个文件的log
  echo 'file filename +p' > /sys/kernel/debug/dynamic_debug/control
  ```

  ```shell
  //打开某个路径的log
  echo 'file dir/* +p' > /sys/kernel/debug/dynamic_debug/control
  ```

- 关闭动态调试

  ```shell
  //和打开对应,+p --> -p
  echo 'file filename -p' > /sys/kernel/debug/dynamic_debug/control
  ```

  

# 4 XBL

## 4.1 Download XBL code 

使用-g xbl 来下载XBL code

```shell
repo init -u ssh://$ID@$MIRROR:29419/manifest.git -b htc -m o-rel_gep_qct8998-muskie.xml -g xbl
repo sync vendor/qcom/boot_images
```

## 4.2 Install Toolchain

* 确认Toolchain版本

  * 在bringup中有描述所需要的版本
  * 执行make 如果为安装Toolchain 会提示所需要的版本

* 下载

  * \\andssd2\and_ssd\andssd_shared\Qualcomm\MSM8998\
  * 或者到Qualcomm 官网下载

* 解压到/pkg/qct/software/llvm/release/arm/

  ```shell
  sudo chown $USER:$USER . -R
  ```

## 4.3 Build XBL

### 4.3.1 openssl 版本过高问题

在编译过程中如果出现

```shell
sectools_builder.py returned non-zero
```

说明openssl版本过高，需要安装低版本的openssl。

* [下载低版本openssl](<https://www.openssl.org/source/old/1.0.2/>)

* 安装

  * 编译安装

    ```shell
    ./config
    make
    make test
    sudo make install
    ```

  * 配置路径

    ```shell
    export PATH=/usr/local/ssl/bin:$PATH
    ```




# 5 Nanohub

## 5.1 download codebase

```shell
git clone ssh://rulei_zhou@tpe.git.htc.com:29419/common/htc/nanohub.git
cd nanohub
git branch -a 
git checkout -b <LOCAL_BRANCH_NAME> <REMOTE_BRANCH>
```

## 5.2 Environment

* [download gcc-arm-none-eabi](https://launchpad.net/gcc-arm-embedded/+download)

* extract (tar jxvf)

* set alias han_setupCHRE

  ```shell
  alias han_setupCHRE='export PATH=$HOME/gcc-arm-none-eabi-5_2-2015q4/bin/:${PATH}; export CROSS_COMPILE=arm-none-eabi-'
  ```

## 5.3 Build

* `han_setupCHRE`

* Check which project you want to build (You may use **ls project**)

* build

  ```shell
  ./build.sh <PROJECT_NAME>
  ```

* flash 

  ```shell
  adb push ./out/nanohub.full.oceannote.bin vendor/firmware/nanohub.full.bin
  adb shell nanoapp_cmd download
  ```

## 5.4 Commit

* master

  ```shell
  git push ssh://rulei_zhou@git.htc.com:29419/common/htc/nanohub.git HEAD:refs/for/sensor/master
  ```

* chre1.0

  ```shell
  git push ssh://rulei_zhou@git.htc.com:29419/common/htc/nanohub.git HEAD:refs/for/sensor/master-chre-1.0
  ```

* Jenkins build

  提交之后Jenkins 会进行编译。编译成功后会有相应fw的链接，然后根据需要将fw cherry pick 到我们需要的branch

* Example

  [示例](http://git.htc.com:8081/#/c/1082913/)

  ![](/5-4-1.png)

