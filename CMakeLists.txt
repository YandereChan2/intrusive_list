cmake_minimum_required(VERSION 3.14)
project(intrusive_list LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(intrusive_list INTERFACE)
target_include_directories(intrusive_list INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})

# 只有作为顶层项目时才构建自测；被 FetchContent / add_subdirectory 当依赖引入时
# 只提供 INTERFACE 目标，不往使用方的构建里塞测试目标。
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    enable_testing()

    add_executable(test_intrusive_list test_intrusive_list.cpp)
    target_link_libraries(test_intrusive_list PRIVATE intrusive_list)
    add_test(NAME test_intrusive_list COMMAND test_intrusive_list)

    if(MSVC)
        target_compile_options(test_intrusive_list PRIVATE /utf-8 /W4)
    else()
        target_compile_options(test_intrusive_list PRIVATE -Wall -Wextra)
    endif()
endif()
