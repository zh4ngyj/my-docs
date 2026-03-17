# CMake tutorial
CMake 是个一个开源的跨平台自动化建构系统。CMake 本身不是构建工具，而是生成构建系统的工具，它生成的构建系统可以使用不同的编译器和工具链。

## 优势
- 跨平台支持：支持多种操作系统和编译器
- 简化配置：CMakeLists.txt 
- 自动化构建：CMake 能够自动检测系统上的库和工具
- 灵活性：支持多种构建类型和配置（如 Debug、Release），并允许用户自定义构建选项和模块

## 基本步骤
1. 编写 CMakeLists.txt文件
2. 生成构建文件
3. 执行构建
![workflow](workflow.png)

## 安装与配置
### 安装
支持的操作系统：
- Windows
- linux
- macOS
- FreeBSD
- OpenBSD
- Solaris
- AIX

安装包地址：https://cmake.org/download/

windows:安装包图形化程序安装

macOS：
~~~bash
# via Homebrew
brew install cmake
~~~

linux：
~~~bash
# Ubuntu
sudo apt install cmake

# Fedora 
sudo dnf install cmake

# Arch Linux
sudo pacman -S cmake
~~~

验证：
~~~bash
# 通过验证版本来确认是否安装成功
cmake --version
~~~

### 配置
**设置源码目录和构建目录**
- 源代码目录：指向包含 CMakeLists.txt 文件的目录
- 构建目录：指向存放生成的构建文件的目录

## CMake 基础
### CMakeLists.txt 文件
CMakeLists.txt 是 CMake 的配置文件，用于定义项目的构建规则、依赖关系、编译选项等。
每个 CMake 项目通常都有一个或多个 CMakeLists.txt 文件。

#### 文件结构和基本语法
CMakeLists.txt 文件使用一系列的 CMake 指令来描述构建过程。常见的指令包括
1、指定 CMake 的最低版本要求：cmake_minimum_required(VERSION <version>)
例如：
`cmake_minimum_required(VERSION 3.10)`

2、定义项目的名称和使用的编程语言：project(<project_name> [<language>...])
例如：
`project(MyProject CXX)`

3、指定要生成的可执行文件和其源文件：add_executable(<target> <source_files>...)
例如：
`add_executable(MyExecutable main.cpp other_file.cpp)`

4、创建一个库（静态库或动态库）及其源文件：add_library(<target> <source_files>...)
例如：
`add_library(MyLibrary STATIC library.cpp)`

5、链接目标文件与其他库：target_link_libraries(<target> <libraries>...)
例如：
`target_link_libraries(MyExecutable MyLibrary)`

6、添加头文件搜索路径：include_directories(<dirs>...)
例如：
`include_directories(${PROJECT_SOURCE_DIR}/include)`

7、设置变量的值：set(<variable> <value>...)
例如：
`set(CMAKE_CXX_STANDARD 11)`

8、设置目标属性：
target_include_directories(TARGET target_name
                          [BEFORE | AFTER]
                          [SYSTEM] [PUBLIC | PRIVATE | INTERFACE]
                          [items1...])
例如：
`target_include_directories(MyExecutable PRIVATE ${PROJECT_SOURCE_DIR}/include)`

9、安装规则：
~~~
install(TARGETS target1 [target2 ...]
        [RUNTIME DESTINATION dir]
        [LIBRARY DESTINATION dir]
        [ARCHIVE DESTINATION dir]
        [INCLUDES DESTINATION [dir ...]]
        [PRIVATE_HEADER DESTINATION dir]
        [PUBLIC_HEADER DESTINATION dir])
~~~
例如：
`install(TARGETS MyExecutable RUNTIME DESTINATION bin)`

10、条件语句 (if, elseif, else, endif 命令)
~~~
if(expression)
  # Commands
elseif(expression)
  # Commands
else()
  # Commands
endif()
~~~
例如：
~~~
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  message("Debug build")
endif()
~~~

11、自定义命令 (add_custom_command 命令)：
~~~
add_custom_command(
   TARGET target
   PRE_BUILD | PRE_LINK | POST_BUILD
   COMMAND command1 [ARGS] [WORKING_DIRECTORY dir]
   [COMMAND command2 [ARGS]]
   [DEPENDS [depend1 [depend2 ...]]]
   [COMMENT comment]
   [VERBATIM]
)
~~~
例如：
~~~
add_custom_command(
   TARGET MyExecutable POST_BUILD
   COMMAND ${CMAKE_COMMAND} -E echo "Build completed."
)
~~~

### 实例
一个简单的 CMakeLists.txt 文件示例：
~~~
cmake_minimum_required(VERSION 3.10)
project(MyProject CXX)

# 添加源文件
add_executable(MyExecutable main.cpp)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
~~~

#### 变量和缓存
CMake 使用变量来存储和传递信息，这些变量可以在 CMakeLists.txt 文件中定义和使用。
变量可以分为普通变量和缓存变量。

**变量定义与使用**
定义变量：
`set(MY_VAR "Hello World")`
使用变量：
`message(STATUS "Variable MY_VAR is ${MY_VAR}")`

**缓存变量:**
缓存变量存储在 CMake 的缓存文件中，用户可以在 CMake 配置时修改这些值。缓存变量通常用于用户输入的设置，例如编译选项和路径。

定义缓存变量：
`set(MY_CACHE_VAR "DefaultValue" CACHE STRING "A cache variable")`

使用缓存变量：
`message(STATUS "Cache variable MY_CACHE_VAR is ${MY_CACHE_VAR}")`

#### 查找库和包
CMake 可以通过 find_package() 指令自动检测和配置外部库和包。
常用于查找系统安装的库或第三方库。


