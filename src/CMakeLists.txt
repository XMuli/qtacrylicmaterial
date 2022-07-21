set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(BUILD_SHARED_LIBS ON)  # 默认生成 .dll

find_package(QT NAMES Qt6 Qt5 COMPONENTS Gui REQUIRED)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Gui REQUIRED)
find_package(QT NAMES Qt6 Qt5 COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Widgets)
find_package(QT NAMES Qt6 Qt5 COMPONENTS Quick)
find_package(Qt${QT_VERSION_MAJOR} COMPONENTS Quick)

set(SOURCES
    qtacrylichelper.qrc
    qtacrylichelper_global.h
    qtacryliceffecthelper.h
    qtacryliceffecthelper.cpp
    utilities.h
    utilities.cpp
)

if(TARGET Qt${QT_VERSION_MAJOR}::Widgets)
    list(APPEND SOURCES
        qtacrylicwidget.h
        qtacrylicwidget.cpp
    )
endif()

if(TARGET Qt${QT_VERSION_MAJOR}::Quick)
    list(APPEND SOURCES
        qtacrylicitem.h
        qtacrylicitem.cpp
    )
endif()

if(WIN32)
    list(APPEND SOURCES
        qtacryliceffecthelper_win32.h
        qtacryliceffecthelper_win32.cpp
        utilities_win32.cpp
    )
    if(BUILD_SHARED_LIBS)                  # 全局变量，默认为 ON，生成动态库；若为 OFF，则生成静态库
        enable_language(RC)                # 更多是使用 MinGW 需要开启。 当一个语言被启用，CMake为该语言查找编译器，决定一些信息，如编译器的供应商和版本，目标架构和位宽，相应工具的位置等
        list(APPEND SOURCES qtacrylichelper.rc)
    endif()
endif()

message("------->" ${PROJECT_NAME})
message("------->" wangwenx190::${PROJECT_NAME})

add_library(${PROJECT_NAME} ${SOURCES})
add_library(wangwenx190::${PROJECT_NAME} ALIAS ${PROJECT_NAME})  # ALIAS 库的别名

target_include_directories(${PROJECT_NAME} PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>"
)

# 指定目标（exe或者so文件） 编译时，要包含的定义， Ref: https://zhuanlan.zhihu.com/p/82244559
target_compile_definitions(${PROJECT_NAME} PRIVATE
    QT_NO_CAST_FROM_ASCII                     # 禁用一切 双引号字符串字面量 传入 QString
    QT_NO_CAST_TO_ASCII
    QT_NO_KEYWORDS                            # 禁止 emit / signals关键字， 可改用 Q_SIGNALS / Q_SLOTS 替换
    QT_DEPRECATED_WARNINGS                    # 若 Qt 的某些功能被标记为过时的，那么编译器会发出警告
    QT_DISABLE_DEPRECATED_BEFORE=0x060000     # 禁用 6.0 版本之前已废弃或过时的方法
    QTACRYLICHELPER_BUILD_LIBRARY             # 自创的一个宏，用在后面 src\qtacrylichelper_global.h 中
)

target_link_libraries(${PROJECT_NAME} PRIVATE
    Qt${QT_VERSION_MAJOR}::GuiPrivate
)

# 存在 目标 Qt5：：Widgets，则添加链接 Qt5：：Widgets
if(TARGET Qt${QT_VERSION_MAJOR}::Widgets)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt${QT_VERSION_MAJOR}::Widgets
    )
endif()

if(TARGET Qt${QT_VERSION_MAJOR}::Quick)
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt${QT_VERSION_MAJOR}::Quick
    )
endif()

if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        WIN32_LEAN_AND_MEAN             # 仅仅包含常用的 API 的头文件，加快编译速度，效果明显
        _CRT_SECURE_NO_WARNINGS         # 允许使用 strcpy，scanf等不安全的函数
        UNICODE
        _UNICODE
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE dwmapi)    # Microsoft Desktop Window Manager API
endif()

if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)
    if(NOT (CMAKE_BUILD_TYPE STREQUAL "Debug"))
        target_compile_options(${PROJECT_NAME} PRIVATE /guard:cf)  # 此处开启； 默认是关闭，
        target_link_options(${PROJECT_NAME} PRIVATE /GUARD:CF)
    endif()
endif()
