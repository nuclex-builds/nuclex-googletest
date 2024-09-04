#!/usr/bin/cmake
cmake_minimum_required (VERSION 3.18)

# -------------------------------------------------------------------------------------------------

project(
  GoogleTest
  VERSION 1.8.1
  DESCRIPTION "Popular C++ unit testing framework by Google"
)

# This sets a bunch of compile flags and defined ${NUCLEX_COMPILER_TAG} to
# say something like linux-gcc-13.2-amd64-debug. You should have this directory
# if you do a full clone of a project that is using this third-party library build.
include("../../build-system/cmake/cplusplus.cmake")

# -------------------------------------------------------------------------------------------------

if(NOT EXISTS ${PROJECT_SOURCE_DIR}/build)
  message(STATUS "Build directory not present, downloading sources")

  file(
    DOWNLOAD https://github.com/google/googletest/archive/release-1.8.1.tar.gz
    ${PROJECT_SOURCE_DIR}/downloads/release-1.8.1.tar.gz
    SHOW_PROGRESS
    EXPECTED_HASH SHA256=9bf1fe5182a604b4135edc1a425ae356c9ad15e9b23f9f12a02e80184c3a249c
    STATUS DOWNLOAD_STATUS
  )
  list(GET DOWNLOAD_STATUS 0 STATUS_CODE)
  if(NOT ${STATUS_CODE} EQUAL 0)
    list(GET DOWNLOAD_STATUS 1 ERROR_MESSAGE)
    list(GET DOWNLOAD_STATUS 2 HTTP_CODE)
    message(
      FATAL_ERROR
      "Could not download GoogleTest sources: ${ERROR_MESSAGE} (HTTP result ${HTTP_CODE})"
    )
  endif()

  message(STATUS "Extracting GoogleTest sources")
  file(
    ARCHIVE_EXTRACT
    INPUT ${PROJECT_SOURCE_DIR}/downloads/release-1.8.1.tar.gz
    DESTINATION ${CMAKE_BINARY_DIR}/extract
  )
  file(
    RENAME
    ${CMAKE_BINARY_DIR}/extract/googletest-release-1.8.1
    ${PROJECT_SOURCE_DIR}/build
  )
endif()

# -------------------------------------------------------------------------------------------------

file(
  GLOB_RECURSE headerFiles
  CONFIGURE_DEPENDS
  "build/googletest/include/*.h"
)
set(
  sourceFiles
  "build/googletest/src/gtest.cc"
  "build/googletest/src/gtest-death-test.cc"
  "build/googletest/src/gtest-filepath.cc"
  "build/googletest/src/gtest-port.cc"
  "build/googletest/src/gtest-printers.cc"
  "build/googletest/src/gtest-test-part.cc"
  "build/googletest/src/gtest-typed-test.cc"
)

# -------------------------------------------------------------------------------------------------

add_library(GoogleTest STATIC)
add_library(GoogleTest::Static ALIAS GoogleTest)

target_include_directories(
  GoogleTest
  PUBLIC "build/googletest/include"
  PRIVATE "build/googletest"
)

target_sources(
  GoogleTest
  PUBLIC ${headerFiles}
  PRIVATE ${sourceFiles}
)

#set_target_properties(GoogleTest PROPERTIES PREFIX "")
set_target_properties(GoogleTest PROPERTIES OUTPUT_NAME "gtest")

# -------------------------------------------------------------------------------------------------

add_library(GoogleTestMain STATIC)
add_library(GoogleTest::Main ALIAS GoogleTestMain)

target_include_directories(
  GoogleTestMain
  PUBLIC "build/googletest/include"
  PRIVATE "build/googletest"
)

target_sources(
  GoogleTestMain
  PRIVATE ${headerFiles}
  PRIVATE "build/googletest/src/gtest_main.cc"
)

#set_target_properties(GoogleTestMain PROPERTIES PREFIX "")
set_target_properties(GoogleTestMain PROPERTIES OUTPUT_NAME "gtest_main")

# -------------------------------------------------------------------------------------------------

install(
  TARGETS GoogleTest
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)
install(
  TARGETS GoogleTestMain
  ARCHIVE DESTINATION ${PROJECT_SOURCE_DIR}/bin/${NUCLEX_COMPILER_TAG}
)
