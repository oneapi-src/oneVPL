# ##############################################################################
# Copyright (C) Intel Corporation
#
# SPDX-License-Identifier: MIT
# ##############################################################################
# oneAPI Video Processing Library (oneVPL) main build script
cmake_minimum_required(VERSION 3.10.2)

file(STRINGS "version.txt" version_txt)
project(oneVPL VERSION ${version_txt})

set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${CMAKE_CURRENT_SOURCE_DIR}/cmake")

#
# Project configuration options
#

# Set default build type to Release if not specified
if(NOT CMAKE_BUILD_TYPE)
  message(STATUS "Default CMAKE_BUILD_TYPE not set using Release")
  set(CMAKE_BUILD_TYPE
      "Release"
      CACHE
        STRING
        "Choose build type from: None Debug Release RelWithDebInfo MinSizeRel"
        FORCE)
endif()

# Project options
set(BUILD_DISPATCHER
    ON
    CACHE BOOL "Build dispatcher.")
set(BUILD_DEV
    ON
    CACHE BOOL "Build files that are only needed for development.")

# Off by default while this is a preview
set(BUILD_PYTHON_BINDING
    OFF
    CACHE BOOL "Build Python binding.")

option(BUILD_DISPATCHER_ONLY "Build dispatcher only." OFF)
option(BUILD_DEV_ONLY "Build only developer package." OFF)
option(BUILD_PYTHON_BINDING_ONLY "Build only Python binding." OFF)

if(BUILD_DISPATCHER_ONLY)
  set(BUILD_DEV OFF)
  set(BUILD_PYTHON_BINDING OFF)
endif()
if(BUILD_DEV_ONLY)
  set(BUILD_DISPATCHER OFF)
  set(BUILD_PYTHON_BINDING OFF)
endif()
if(BUILD_PYTHON_BINDING_ONLY)
  set(BUILD_DEV OFF)
  set(BUILD_DISPATCHER OFF)
endif()

option(USE_ONEAPI_INSTALL_LAYOUT "Use oneAPI install layout instead of FHS" OFF)

if(USE_ONEAPI_INSTALL_LAYOUT)
  set(CMAKE_INSTALL_DOCDIR "documentation")
  set(CMAKE_INSTALL_LIBDIR "lib")
  set(ONEAPI_INSTALL_ENVDIR "env")
  set(ONEAPI_INSTALL_SYSCHECKDIR "sys_check")
  set(ONEAPI_INSTALL_MODFILEDIR "modulefiles")
  set(ONEAPI_INSTALL_EXAMPLEDIR "examples")
  set(ONEAPI_INSTALL_LICENSEDIR "licensing/oneVPL")
  set(ONEAPI_INSTALL_PYTHONDIR "python/lib")
endif()

# Set output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

include(cmake/oneAPIInstallDirs.cmake)

# Keep executable bits on shared objects when installing regardless of distro
set(CMAKE_INSTALL_SO_NO_EXE 0)

include(cmake/CompileOptions.cmake)

add_subdirectory(api)
add_subdirectory(dispatcher)
if(BUILD_DEV)
  add_subdirectory(env)
  add_subdirectory(modulefiles)
  add_subdirectory(tools)
  add_subdirectory(examples)
endif()
add_subdirectory(preview)

install(
  FILES third-party-programs.txt
  DESTINATION ${ONEAPI_INSTALL_LICENSEDIR}
  COMPONENT license)

install(
  FILES LICENSE
  RENAME license.txt
  DESTINATION ${ONEAPI_INSTALL_LICENSEDIR}
  COMPONENT license)

if(USE_ONEAPI_INSTALL_LAYOUT)
  install(
    FILES LICENSE
    RENAME license.txt
    DESTINATION "licensing"
    COMPONENT license)
endif()

include(cmake/PackageTarget.cmake)

message(
  STATUS "---------------- Configuration summary ------------------------------"
)
message(STATUS "CMake:")
message(STATUS "  CMAKE_VERSION                  : ${CMAKE_VERSION}")
message(STATUS "  CMAKE_GENERATOR                : ${CMAKE_GENERATOR}")
message(STATUS "  CMAKE_BUILD_TOOL               : ${CMAKE_BUILD_TOOL}")
message(STATUS "Target:")
message(STATUS "  CMAKE_SYSTEM_NAME              : ${CMAKE_SYSTEM_NAME}")
message(STATUS "  CMAKE_SYSTEM_VERSION           : ${CMAKE_SYSTEM_VERSION}")
message(STATUS "  CMAKE_SYSTEM_PROCESSOR         : ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "General:")
message(STATUS "  CMAKE_BUILD_TYPE               : ${CMAKE_BUILD_TYPE}")
message(STATUS "  CMAKE_TOOLCHAIN_FILE           : ${CMAKE_TOOLCHAIN_FILE}")
message(STATUS "  CMAKE_C_COMPILER               : ${CMAKE_C_COMPILER}")
message(STATUS "  CMAKE_CXX_COMPILER             : ${CMAKE_CXX_COMPILER}")
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  message(STATUS "  Build architecture             : 64-bit")
elseif(CMAKE_SIZEOF_VOID_P EQUAL 4)
  message(STATUS "  Build architecture             : 32-bit")
endif()
message(STATUS "Flags:")
message(STATUS "  CMAKE_C_FLAGS                  : ${CMAKE_C_FLAGS}")
message(STATUS "  CMAKE_CXX_FLAGS                : ${CMAKE_CXX_FLAGS}")
message(STATUS "Release flags:")
message(STATUS "  CMAKE_C_FLAGS_RELEASE          : ${CMAKE_C_FLAGS_RELEASE}")
message(STATUS "  CMAKE_CXX_FLAGS_RELEASE        : ${CMAKE_CXX_FLAGS_RELEASE}")
message(STATUS "Debug flags:")
message(STATUS "  CMAKE_C_FLAGS_DEBUG            : ${CMAKE_C_FLAGS_DEBUG}")
message(STATUS "  CMAKE_CXX_FLAGS_DEBUG          : ${CMAKE_CXX_FLAGS_DEBUG}")
message(STATUS "RelWithDebRT flags:")
message(
  STATUS "  CMAKE_C_FLAGS_RELWITHDEBRT     : ${CMAKE_C_FLAGS_RELWITHDEBRT}")
message(
  STATUS "  CMAKE_CXX_FLAGS_RELWITHDEBRT   : ${CMAKE_CXX_FLAGS_RELWITHDEBRT}")
message(STATUS "Install:")
message(STATUS "  CMAKE_INSTALL_PREFIX           : ${CMAKE_INSTALL_PREFIX}")
message(
  STATUS "  CMAKE_INSTALL_FULL_BINDIR      : ${CMAKE_INSTALL_FULL_BINDIR}")
message(
  STATUS "  CMAKE_INSTALL_FULL_INCLUDEDIR  : ${CMAKE_INSTALL_FULL_INCLUDEDIR}")
message(
  STATUS "  CMAKE_INSTALL_FULL_LIBDIR      : ${CMAKE_INSTALL_FULL_LIBDIR}")
message(
  STATUS "  CMAKE_INSTALL_FULL_DOCDIR      : ${CMAKE_INSTALL_FULL_DOCDIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_BIN32DIR   : ${ONEAPI_INSTALL_FULL_BIN32DIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_LIB32DIR   : ${ONEAPI_INSTALL_FULL_LIB32DIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_ENVDIR     : ${ONEAPI_INSTALL_FULL_ENVDIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_SYSCHECKDIR: ${ONEAPI_INSTALL_FULL_SYSCHECKDIR}"
)
message(
  STATUS "  ONEAPI_INSTALL_FULL_MODFILEDIR : ${ONEAPI_INSTALL_FULL_MODFILEDIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_EXAMPLEDIR : ${ONEAPI_INSTALL_FULL_EXAMPLEDIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_LICENSEDIR : ${ONEAPI_INSTALL_FULL_LICENSEDIR}")
message(
  STATUS "  ONEAPI_INSTALL_FULL_PYTHONDIR  : ${ONEAPI_INSTALL_FULL_PYTHONDIR}")
message(STATUS "Build:")
message(STATUS "  BUILD_DEV                      : ${BUILD_DEV}")
message(STATUS "  BUILD_DISPATCHER               : ${BUILD_DISPATCHER}")
