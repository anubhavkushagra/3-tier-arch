cmake_minimum_required(VERSION 3.10)
project(i3_load_client)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- Find Packages ---
find_package(PkgConfig REQUIRED)
pkg_check_modules(GRPC REQUIRED grpc++)
pkg_check_modules(PROTOBUF REQUIRED protobuf)
find_package(Threads REQUIRED)
find_package(OpenSSL REQUIRED)

find_program(PROTOC_EXECUTABLE protoc REQUIRED)
find_program(GRPC_CPP_PLUGIN_EXECUTABLE grpc_cpp_plugin REQUIRED)

# --- Generate Protobuf and gRPC files from .proto ---
set(PROTO_SRC "${CMAKE_CURRENT_SOURCE_DIR}/proto/kv.proto")
set(GEN_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${GEN_DIR})

add_custom_command(
    OUTPUT "${GEN_DIR}/kv.pb.cc" "${GEN_DIR}/kv.pb.h" "${GEN_DIR}/kv.grpc.pb.cc" "${GEN_DIR}/kv.grpc.pb.h"
    COMMAND ${PROTOC_EXECUTABLE} --cpp_out=${GEN_DIR} --grpc_out=${GEN_DIR} --plugin=protoc-gen-grpc=${GRPC_CPP_PLUGIN_EXECUTABLE} -I${CMAKE_CURRENT_SOURCE_DIR}/proto ${PROTO_SRC}
    DEPENDS ${PROTO_SRC}
)

include_directories(${GEN_DIR} ${GRPC_INCLUDE_DIRS} ${PROTOBUF_INCLUDE_DIRS})

# --- Client Executable ---
add_executable(distributed_load_client 
    distributed_load_client.cpp
    "${GEN_DIR}/kv.pb.cc"
    "${GEN_DIR}/kv.grpc.pb.cc"
)

target_link_libraries(distributed_load_client ${GRPC_LIBRARIES} ${PROTOBUF_LIBRARIES} Threads::Threads)
target_link_libraries(distributed_load_client grpc++ grpc protobuf OpenSSL::SSL OpenSSL::Crypto)
