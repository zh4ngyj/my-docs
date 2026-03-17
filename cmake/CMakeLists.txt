# 一个简单的 CMakeLists.txt 文件示例
# 指定 CMake 的最低版本要求
cmake_minimum_required(VERSION 3.10)

# 定义项目的名称和使用的编程语言
project(myproject CXX)

# 添加源文件
add_executable(MyExecutable main.cpp)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
