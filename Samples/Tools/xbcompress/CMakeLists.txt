﻿# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.
#
# This is a development only tool. It is not supported for use on retail kits.

cmake_minimum_required (VERSION 3.15)

project(xbcompress
  DESCRIPTION "Microsoft SZDD/KWAJ-style compression tool for Windows & Xbox"
  LANGUAGES CXX)

if(NOT WIN32)
   message(FATAL_ERROR "This tool is compatible with Windows 10 and Xbox")
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 4)
    message(FATAL_ERROR "ERROR: This tool requires 64-bit")
endif()

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

# Always use retail static CRT for this tool
set(CMAKE_MSVC_RUNTIME_LIBRARY MultiThreaded)

add_executable(${PROJECT_NAME} compresstool.cpp)

# Use Warning Level 4
string(REPLACE "/W3 " "/W4 " CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
string(REPLACE "/W3 " "/W4 " CMAKE_CXX_FLAGS_DEBUG ${CMAKE_CXX_FLAGS_DEBUG})
string(REPLACE "/W3 " "/W4 " CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})

# Disable RTTI
string(REPLACE "/GR " "/GR- " CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
string(REPLACE "/GR " "/GR- " CMAKE_CXX_FLAGS_DEBUG ${CMAKE_CXX_FLAGS_DEBUG})
string(REPLACE "/GR " "/GR- " CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})

target_compile_definitions(${PROJECT_NAME} PRIVATE _CONSOLE _UNICODE UNICODE _WIN32_WINNT=0x0A00)

target_compile_options(${PROJECT_NAME} PRIVATE /fp:fast /GS /Gy)

# Don't link with kernel32.lib, etc.
set(CMAKE_CXX_STANDARD_LIBRARIES "")
set(CMAKE_CXX_STANDARD_LIBRARIES_INIT "")

# Not all APIs in onecore_apiset.lib are in xgameplatform.lib, but there's
# sufficent overlap for this tool.
target_link_libraries(${PROJECT_NAME} PRIVATE onecore_apiset.lib)

target_link_options(${PROJECT_NAME} PRIVATE /SUBSYSTEM:CONSOLE,10.0 /NODEFAULTLIB:kernel32.lib /NODEFAULTLIB:onecore.lib)

if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
   target_compile_options(${PROJECT_NAME} PRIVATE /permissive- /Zc:__cplusplus)

   if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 19.24)
      target_compile_options(${PROJECT_NAME} PRIVATE /ZH:SHA_256)
   endif()

   if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 19.26)
      target_compile_options(${PROJECT_NAME} PRIVATE /Zc:preprocessor)
   endif()

   if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 19.27)
      target_link_options(${PROJECT_NAME} PRIVATE /CETCOMPAT)
   endif()
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
   target_compile_options(${PROJECT_NAME} PRIVATE
     -Wno-c++98-compat
     -Wno-c++98-compat-pedantic
     -Wno-gnu-anonymous-struct
     -Wno-language-extension-token
     -Wno-nested-anon-types
     -Wno-reserved-id-macro
     -Wno-unknown-pragmas)
endif()

# Deploy using:
#   xbcp bin\*.exe xd:\
#
# Run using:
#   xbrun /O D:\xbcompress.exe
