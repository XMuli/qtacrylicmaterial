cmake_minimum_required(VERSION 3.19)

project(QtAcrylicMaterial LANGUAGES CXX)

option(BUILD_EXAMPLES "Build QtAcrylicMaterial demo applications." ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(NOT CMAKE_DEBUG_POSTFIX)
    if(WIN32)
        set(CMAKE_DEBUG_POSTFIX d)
    else()
        set(CMAKE_DEBUG_POSTFIX _debug)
    endif()
endif()

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

if(NOT (CMAKE_BUILD_TYPE STREQUAL "Debug"))
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
endif()

#           动态库      静态库           # Ref： https://blog.csdn.net/liuzehn/article/details/78424972
# Windows    .dll       .lib
# Linux      .so        .a
# Mac        .dylib     .a             (.framework 即可能是动态库、也可能是静态库)

if(NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)   # 设置 可执行文件 目标的输出路径
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
endif()

if(NOT CMAKE_LIBRARY_OUTPUT_DIRECTORY)   # 设置动态库的输出目录
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
endif()

if(NOT CMAKE_ARCHIVE_OUTPUT_DIRECTORY)   # 设置静态库的输出目录
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
endif()

add_subdirectory(src)
if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()
