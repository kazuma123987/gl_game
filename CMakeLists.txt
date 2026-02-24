cmake_minimum_required(VERSION 3.12)
set(CMAKE_SYSTEM_PROCESSOR AMD64)
include(scripts/arch.cmake)

# 设置项目名和输出目录
project(main
  VERSION 0.0.1
  LANGUAGES C CXX)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${CMAKE_SOURCE_DIR}")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# 设置头文件目录(如果设置为源目录程序将改为#include "include/GLFW/glfw3.h")
include_directories(${CMAKE_SOURCE_DIR}/include/tools
  ${CMAKE_SOURCE_DIR}/include/user
  ${CMAKE_SOURCE_DIR}/src/tools/freetype/include
  ${CMAKE_SOURCE_DIR}/include/tools/fmod/2.03.12)

# 添加源文件
set(GLAD "${CMAKE_SOURCE_DIR}/src/tools/glad/glad.c")
file(GLOB SOURCES "${CMAKE_SOURCE_DIR}/src/user/*.cpp")

link_directories(${CMAKE_SOURCE_DIR}/lib/fmod/2.03.12/${ARCH})

if(USE_GCC)
  set(FMOD_LIB ${CMAKE_SOURCE_DIR}/fmod.dll)
  set(FMODL_LIB ${CMAKE_SOURCE_DIR}/fmodL.dll)
elseif(USE_CLANG)
  # 不支持的编译器，抛出错误
  message(FATAL_ERROR "Unsupported compiler: ${CMAKE_CXX_COMPILER_ID}")
elseif(USE_VS20XX)
  set(FMOD_LIB fmod_vc)
  set(FMODL_LIB fmodL_vc)
endif()

add_subdirectory(src/tools/glfw3)

# 禁用 FreeType 中编译有问题的模块
# option(FT_DISABLE_BZIP2 "Disable BZIP2 driver" ON)
# option(FT_DISABLE_HARFBUZZ "Disable HarfBuzz driver" ON)
# option(FT_DISABLE_PNG "Disable PNG support" ON)
# option(FT_DISABLE_ZLIB "Disable ZLIB support" ON)
# option(FT_DISABLEBROTLI "Disable BROTLI support" ON)
add_subdirectory(src/tools/freetype)

if("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
  # 设置版本、文件说明、文件名称、版权信息(自定义部分)
  set(VERSION_MAJOR 0)
  set(VERSION_MINOR 0)
  set(VERSION_PATCH 0)
  set(VERSION_BUILD 1)
  set(FILE_DESCRIPTION "my game demo")
  set(PRODUCT_NAME "game")
  set(COPYRIGHT "Copyright (C) 2023")

  # 把工作目录下的version.rc.in编译到build目录下的version.rc
  configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/res/icon/version.rc.in
    ${CMAKE_CURRENT_BINARY_DIR}/version.rc
    @ONLY)
  add_executable(${PROJECT_NAME} ${SOURCES} ${GLAD} ${CMAKE_CURRENT_BINARY_DIR}/version.rc) # 添加可执行文件
else()
  add_executable(${PROJECT_NAME} ${SOURCES} ${GLAD}) # 添加可执行文件
endif()

# 链接 FMOD 和 GLFW 库（根据需要修改库文件的路径）
if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
  # target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/fmodL.dll)
  target_link_libraries(${PROJECT_NAME} PRIVATE ${FMODL_LIB})
else()
  # target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/fmod.dll)
  target_link_libraries(${PROJECT_NAME} PRIVATE ${FMOD_LIB})
endif()

target_link_libraries(${PROJECT_NAME} PRIVATE glfw freetype)
