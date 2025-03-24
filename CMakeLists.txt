project(libunwind)

cmake_minimum_required(VERSION 3.16.1)

set(CMAKE_C_STANDARD 11)

set(PKG_MAJOR "1")
set(PKG_MINOR "6")
set(PKG_EXTRA "-rc2")
set(PACKAGE_STRING "libunwind")
set(PACKAGE_BUGREPORT "")

set(CMAKE_COMPILE_WARNING_AS_ERROR ON)

# Android target detection
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
    add_definitions(-DNO_LINK_H)
    add_definitions(-DHAVE_DL_ITERATE_PHDR=0)
    add_definitions(-DHAVE_MMAP=0)
    
    # Android-specific configurations
    set(HAVE_ELF_H 1)
    set(HAVE_ENDIAN_H 0)
    set(HAVE_STDALIGN_H 1)
    set(HAVE_STDATOMIC_H 1)
    set(HAVE_THREAD_LOCAL 1)
elseif("${CMAKE_GENERATOR}" MATCHES "^Visual Studio.*$")
    # Original Visual Studio configurations here
    # ... [keep all the existing Visual Studio specific code]
else()
    message(FATAL_ERROR "This CMake file is currently only designed for building on Android or Visual Studio")
endif()

add_definitions(-DHAVE_CONFIG_H)

configure_file(include/config.h.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/include/config.h)
configure_file(include/libunwind-common.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/libunwind-common.h)
configure_file(include/libunwind.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/libunwind.h)
configure_file(include/tdep/libunwind_i.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/tdep/libunwind_i.h)

add_subdirectory(src)
