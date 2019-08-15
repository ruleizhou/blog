---
title: git patch
date: 2019-08-15 14:11:40
categories:
- tools
tags:
- tools
- git
---

# 概述

Git提供两种补丁方案：

- UNIX标准补丁.diff文件

  只是记录文件改变的内容，不带有commit记录信息,多个commit可以合并成一个diff文件

- Git专用.patch文件

  带有记录文件改变的内容，也带有commit记录信息,每个commit对应一个patch文件



# 创建

## 创建patch

- 创建某次提交(含)之前的几次提交

  ```shell
  git format-patch 【commit sha1 id】-n
  ```

- 某个提交的patch

  ```shell
  git format-patch 【commit sha1 id】 -1
  ```

- 某两次提交之间的所有patch

  ```shell
  git format-patch 【commit sha1 id】..【commit sha1 id】 
  ```

## 创建diff

```shell
git diff  【commit sha1 id】 【commit sha1 id】 >  【diff文件名】
```



# 应用

## 检查patch文件

```shell
git apply --stat 【path/to/xxx.patch】
```

## 查看是否能正常打入

```shell
git apply --check 【path/to/xxx.patch】
```

## 应用patch

```shell
git  am 【path/to/xxx.patch】
```



# 解决冲突

先使用git am 打入patch，若是存在冲突部分，git会拷贝patch到

```shell
.git/rebase-apply/
```

然后在使用上述路径中的patch进行如下操作。

- 自动合入patch中不冲突的部分，同时保留冲突的部分

  ```shell
  git  apply --reject  .git/rebase-apply/0001
  ```

  - 会生成后缀为.rej的文件（未合入的冲突部分）
  - 根据.rej文件，手动合入
  - 解决完冲突后删除后缀为 .rej 的文件。

- 将文件添加到缓存区

  ```shell
  git add filename
  ```

- 重新打入patch

  ```shell
  git am --continue
  ```

