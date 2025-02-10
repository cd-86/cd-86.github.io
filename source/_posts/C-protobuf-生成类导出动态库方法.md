---
title: C++ protobuf 生成类导出动态库方法
date: 2025-02-10 17:04:20
tags: [CMake, C++, Protobuf]
---

*protobuf 版本 3.6.1*

1. 修改 `protoc` 代码 增加 `include` 导出头文件

    需要给生成的 `proto` 类文件增加 引用头文件 需要对以下文件做出修改 增加include参数

    a. 自行定义导出头文件
        
    a. 对 `src/google/protobuf/compiler/cpp/cpp_generator.cc` 的 82 行增加
    ```cpp
        //    ---snip---
        } else if (options[i].first == "include") {
            file_options.include = options[i].second;
        //    ---snip---
    ```
    b. 对 `src/google/protobuf/compiler/cpp/cpp_file.cc` 的 1249 行增加
    ```cpp
        //    ---snip---
        if (!options_.include.empty()) {
            printer->Print("#include <$head$>\n","head",options_.include);
        }
        //    ---snip---
    ```
   	c. 编译 `protoc`
    d. 执行生成命令
    ```bash
    protoc.exe -I=%SRC_DIR% --cpp_out=dllexport_decl=XX_EXPORT,include=XX_Global.h:%DST_DIR% %SRC_DIR%
    ```
    e. 在 `ProjectName ` 的 `CMakeLists.txt` 定义导出头文件的宏

2. 复用 PROTOC 的导出宏

    a. 执行生成命令
    ```bash
        protoc.exe -I=%SRC_DIR% --cpp_out=dllexport_decl=PROTOC_EXPORT:%DST_DIR% %SRC_DIR%
    ```
    b. 在 `ProjectName ` 的 `CMakeLists.txt` 定义如下宏
    ```bash 
        target_compile_definitions(ProjectName PRIVATE LIBPROTOC_EXPORTS)
    ```
在导出的目录 增加一个 `CMakeLists.txt`

```bash
project(ProjectName)

file(GLOB_RECURSE SRCS *.cc *.cpp *.c)
file(GLOB_RECURSE HRDS *.h)

add_library(ProjectName SHARED ${SRCS} ${HRDS})

target_link_libraries(ProjectName PUBLIC protobuf::libprotobuf)

target_include_directories(ProjectNamePUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)

# 方法2
target_compile_definitions(ProjectName PRIVATE LIBPROTOC_EXPORTS)
```

---
上面 第二种方法可以直接都写在 `CMakeLists.txt` 里
```bash
project(ProjectName)

# 不要递归
file(GLOB PROTO_FILES proto/*.proto)
# 暂时只考虑 windows
set(PROTOC_EXECUTABLE ${CMAKE_SOURCE_DIR}/protoc.exe)
set(PROTO_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/proto)
# 输出到 build 的同级目录下
set(PROTO_DST_DIR ${CMAKE_CURRENT_BINARY_DIR})

set(PROTO_SRCS)
set(PROTO_HRDS)

# 遍历所有 proto 文件 生成 类文件
foreach (proto_file ${PROTO_FILES})
    get_filename_component(proto_file_name ${proto_file} NAME_WE)
    set(output_h ${CMAKE_CURRENT_BINARY_DIR}/${proto_file_name}.pb.h)
    set(output_cc ${CMAKE_CURRENT_BINARY_DIR}/${proto_file_name}.pb.cc)

    add_custom_command(OUTPUT ${output_h} ${output_cc}
            COMMAND ${PROTOC_EXECUTABLE}
            ARGS -I=${PROTO_SRC_DIR}
            --cpp_out=dllexport_decl=PROTOC_EXPORT:${PROTO_DST_DIR}
            ${proto_file}

            DEPENDS ${proto_file}
    )
    list(APPEND PROTO_SRCS ${output_cc})
    list(APPEND PROTO_HRDS ${output_h})
endforeach ()

add_library(ProjectName SHARED ${PROTO_SRCS} ${PROTO_HRDS})

set_source_files_properties(${PROTO_SRCS} ${PROTO_HRDS} PROPERTIES GENERATED TRUE)

target_link_libraries(ProjectName PUBLIC protobuf::libprotobuf)

target_include_directories(ProjectName PUBLIC $<BUILD_INTERFACE:${PROTO_DST_DIR}>)

target_compile_definitions(ProjectName PRIVATE LIBPROTOC_EXPORTS)

```