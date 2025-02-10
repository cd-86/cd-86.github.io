---
title: CLion 配置 Qt 开发环境
date: 2025-02-10 17:08:14
tags:
---

## 	添加外部工具
文件->设置->工具->外部工具
1.  QtDesigner

程序：`C:\Qt\6.7.2\msvc2019_64\bin\designer.exe`
实参：`$FileName$`
工作目录：`$FileDir$`

![img](image.png)

2. UIC

程序：`C:\Qt\6.7.2\msvc2019_64\bin\uic.exe`
实参：`$FileName$ -o ui_$FileNameWithoutExtension$.h`
工作目录：`$FileDir$`

![img1](image-1.png)

## CMake配置
文件->设置->构建、执行、部署->CMake

CMake选项: `-DCMAKE_PREFIX_PATH=C:\Qt\6.7.2\msvc2019_64`

![img2](image-2.png)

## 运行/调试配置
运行->编辑配置

工作目录：`C:\Qt\6.7.2\msvc2019_64\bin`
环境变量：`QT_ASSUME_STDERR_HAS_CONSOLE=1 `

![img3](image-3.png)