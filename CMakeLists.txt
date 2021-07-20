# Copyright (c) 2020 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.19.0 FATAL_ERROR)

set(MBED_PATH ${CMAKE_CURRENT_SOURCE_DIR}/mbed-os CACHE INTERNAL "")
set(MBED_CONFIG_PATH ${CMAKE_CURRENT_BINARY_DIR} CACHE INTERNAL "")
set(APP_TARGET mbed-os-example-blockdevice)

include(${MBED_PATH}/tools/cmake/app.cmake)

project(${APP_TARGET})

add_subdirectory(${MBED_PATH})

add_executable(${APP_TARGET})

mbed_configure_app_target(${APP_TARGET})

target_sources(${APP_TARGET}
    PRIVATE
        main.cpp
)

list(APPEND mbed_storage_libs)

if("DATAFLASH" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-dataflash)
endif()

if("FLASHIAP" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-flashiap)
endif()

if("I2CEE" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-i2cee)
endif()

if("OSPIF" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-ospif)
endif()

if("QSPIF" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-qspif)
endif()

if("SD" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-sd)
endif()

if("SPIF" IN_LIST MBED_TARGET_LABELS)
    list(APPEND mbed_storage_libs mbed-storage-spif)
endif()

target_link_libraries(${APP_TARGET}
    PRIVATE
        mbed-os
        mbed-storage
        ${mbed_storage_libs}
)

mbed_set_post_build(${APP_TARGET})

option(VERBOSE_BUILD "Have a verbose build process")
if(VERBOSE_BUILD)
    set(CMAKE_VERBOSE_MAKEFILE ON)
endif()
