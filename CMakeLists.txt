cmake_minimum_required (VERSION 3.1.3 FATAL_ERROR)
project (etcd-cpp-api)

find_library(CPPREST_LIB NAMES cpprest)
find_path(CPPREST_INCLUDE_DIR NAMES cpprest/http_client.h)

find_package(Boost REQUIRED COMPONENTS system thread locale random)

set (etcd-cpp-api_VERSION_MAJOR 0)
set (etcd-cpp-api_VERSION_MINOR 1)

set(CATCH_INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR})

set(CMAKE_CXX_CLANG_TIDY "")

enable_testing()
include_directories(SYSTEM ${CPPREST_INCLUDE_DIR} ${Boost_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic -Werror -std=c++11")

add_subdirectory(proto)
add_subdirectory(src)
add_subdirectory(tst)
