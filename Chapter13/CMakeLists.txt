cmake_minimum_required(VERSION 3.15)

project(
  ch13
  VERSION 0.0.1
  LANGUAGES CXX)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
list(APPEND CMAKE_PREFIX_PATH "${CMAKE_BINARY_DIR}")

include(CommonCompileFlags)

# find_package(cpprestsdk REQUIRED)
find_package(grpc REQUIRED)
# find_package(openssl REQUIRED)
find_package(protobuf REQUIRED)
# find_package(Threads REQUIRED)

include(protobuf-generate)
find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin REQUIRED)

add_library(grpc_service grpc/service.proto)
target_link_libraries(grpc_service PUBLIC protobuf::libprotobuf grpc::grpc)

protobuf_generate(TARGET grpc_service LANGUAGE cpp)
protobuf_generate(
  TARGET
  grpc_service
  LANGUAGE
  grpc
  GENERATE_EXTENSIONS
  .grpc.pb.h
  .grpc.pb.cc
  PLUGIN
  "protoc-gen-grpc=${GRPC_CPP_PLUGIN}")

target_include_directories(grpc_service PUBLIC "${CMAKE_CURRENT_BINARY_DIR}")

add_executable(grpc_client grpc/AsyncClient.cpp)
target_link_libraries(grpc_client grpc_service)

add_executable(grpc_server grpc/AsyncServer.cpp)
target_link_libraries(grpc_server grpc_service)

# add_executable(rest_client rest/CppRestSdkReadPage.cpp)
# target_link_libraries(rest_client PRIVATE cpprestsdk::cpprest Threads::Threads
# openssl::openssl)
#
# add_executable(rest_server rest/CppRestSdkSimpleService.cpp)
# target_link_libraries(rest_server PRIVATE cpprestsdk::cpprest Threads::Threads
# openssl::openssl)
