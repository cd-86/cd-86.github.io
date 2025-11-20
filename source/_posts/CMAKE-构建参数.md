---
title: CMAKE 构建参数
date: 2025-11-20 11:46:00
tags: [CMake, C++]
---


``` bash
# 创建 build 目录
mkdir build
# 进入 build 目录
cd build
# 运行 CMake
cmake ..
# 指定生成器
cmake -G Ninja -DCMAKE_BUILD_TYPE=Debug -DCMAKE_MAKE_PROGRAM=ninja ..
# 构建项目
cmake --build . --config Debug
cmake --build . --config Release
```
