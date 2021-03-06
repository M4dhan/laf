# LAF Base Library
# Copyright (c) 2001-2017 David Capello

include(CheckIncludeFiles)
include(CheckFunctionExists)
include(TestBigEndian)

check_include_files(stdint.h HAVE_STDINT_H)
check_include_files(dlfcn.h HAVE_DLFCN_H)
check_function_exists(sched_yield HAVE_SCHED_YIELD)

test_big_endian(LAF_BIG_ENDIAN)
if(NOT LAF_BIG_ENDIAN)
  set(LAF_LITTLE_ENDIAN TRUE)
endif()

# Generate config.h file
configure_file(${CMAKE_CURRENT_LIST_DIR}/config.h.cmakein
               ${LAF_BINARY_DIR}/base/config.h @ONLY)

# To include "base/config.h"
include_directories(${LAF_BINARY_DIR})

set(BASE_SOURCES
  base64.cpp
  cfile.cpp
  chrono.cpp
  convert_to.cpp
  debug.cpp
  dll.cpp
  errno_string.cpp
  exception.cpp
  file_handle.cpp
  fs.cpp
  launcher.cpp
  log.cpp
  mem_utils.cpp
  memory.cpp
  memory_dump.cpp
  mutex.cpp
  process.cpp
  program_options.cpp
  replace_string.cpp
  serialization.cpp
  sha1.cpp
  sha1_rfc3174.c
  split_string.cpp
  string.cpp
  system_console.cpp
  thread.cpp
  time.cpp
  version.cpp)

if(APPLE)
  set(BASE_SOURCES ${BASE_SOURCES}
    fs_osx.mm)
endif()

if(WIN32)
  set(BASE_SOURCES ${BASE_SOURCES}
    win/registry.cpp
    win/win32_exception.cpp)
endif()

add_library(laf-base ${BASE_SOURCES})
target_link_libraries(laf-base modpbase64)

# To #include "base/base.h" and "base/config.h"
target_include_directories(laf-base PUBLIC
  ${LAF_LIST_DIR} ${LAF_BINARY_DIR})

if(WIN32)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)
  target_link_libraries(laf-base dbghelp shlwapi)
else()
  find_library(DL_LIBRARY NAMES dl)
  if(DL_LIBRARY)
    target_link_libraries(laf-base ${DL_LIBRARY})
  endif()
endif()

if(LAF_WITH_TESTS)
  laf_find_tests(. laf-base)
  if(WIN32)
    laf_find_tests(win laf-base)
  endif()
endif()
