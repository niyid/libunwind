cmake_minimum_required(VERSION 3.16.1)
project(libunwind)

set(CMAKE_C_STANDARD 11)

set(PKG_MAJOR "1")
set(PKG_MINOR "6")
set(PKG_EXTRA "-rc2")
set(PACKAGE_STRING "libunwind")
set(PACKAGE_BUGREPORT "")

set(CMAKE_COMPILE_WARNING_AS_ERROR ON)

# Add this near the top, after project() declaration
if(ANDROID)
    add_definitions(-DNO_LINK_H)
    set(HAVE_LINK_H 0)
    set(HAVE_MMAP 0)
endif()

# In your architecture detection section, add:
if(ANDROID_ABI STREQUAL "armeabi-v7a")
    add_definitions(-D__arm__)
    set(TARGET_ARM 1)
    set(arch arm)
elseif(ANDROID_ABI STREQUAL "arm64-v8a")
    add_definitions(-D__aarch64__)
    set(TARGET_AARCH64 1)
    set(arch aarch64)
endif()

# Ensure we are targeting Android
if (ANDROID)
  message(STATUS "Building for Android")
else()
  message(FATAL_ERROR "This CMake file is only designed for Android builds")
endif()

# Android target architecture detection
if(ANDROID)
    if(ANDROID_ABI STREQUAL "armeabi-v7a")
        set(TARGET_ARM 1)
        set(arch arm)
        add_definitions(-D__arm__)
    elseif(ANDROID_ABI STREQUAL "arm64-v8a")
        set(TARGET_AARCH64 1)
        set(arch aarch64)
        add_definitions(-D__aarch64__)
    elseif(ANDROID_ABI STREQUAL "x86")
        set(TARGET_X86 1)
        set(arch x86)
        add_definitions(-D__i386__)
    elseif(ANDROID_ABI STREQUAL "x86_64")
        set(TARGET_AMD64 1)
        set(arch x86_64)
        add_definitions(-D__x86_64__)
    else()
        message(FATAL_ERROR "Unsupported Android ABI: ${ANDROID_ABI}")
    endif()
    
    add_definitions(-D__linux__)
    add_definitions(-D__ANDROID__)
else()
    message(FATAL_ERROR "This CMake file is designed for Android builds only")
endif()

# Define common macros
add_definitions(-D__linux__ -DHAVE_CONFIG_H)

# Include directories
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)

# Configure header files
configure_file(include/config.h.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/include/config.h)
configure_file(include/libunwind-common.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/libunwind-common.h)
configure_file(include/libunwind.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/libunwind.h)
configure_file(include/tdep/libunwind_i.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/tdep/libunwind_i.h)

# Add source directory
add_subdirectory(src)
