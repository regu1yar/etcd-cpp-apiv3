find_package(Protobuf REQUIRED)
find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin REQUIRED) # Get full path to plugin

function(PROTOBUF_GENERATE_GRPC_CPP SRCS HDRS)
    if(NOT ARGN)
        message(SEND_ERROR "Error: PROTOBUF_GENERATE_GRPC_CPP() called without any proto files")
        return()
    endif()

    if(PROTOBUF_GENERATE_CPP_APPEND_PATH) # This variable is common for all types of output.
        # Create an include path for each file specified
        foreach(FIL ${ARGN})
            get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
            get_filename_component(ABS_PATH ${ABS_FIL} PATH)
            list(FIND _protobuf_include_path ${ABS_PATH} _contains_already)
            if(${_contains_already} EQUAL -1)
                list(APPEND _protobuf_include_path -I ${ABS_PATH})
            endif()
        endforeach()
    else()
        set(_protobuf_include_path -I ${CMAKE_CURRENT_SOURCE_DIR})
    endif()

    if(DEFINED PROTOBUF_IMPORT_DIRS)
        foreach(DIR ${Protobuf_IMPORT_DIRS})
            get_filename_component(ABS_PATH ${DIR} ABSOLUTE)
            list(FIND _protobuf_include_path ${ABS_PATH} _contains_already)
            if(${_contains_already} EQUAL -1)
                list(APPEND _protobuf_include_path -I ${ABS_PATH})
            endif()
        endforeach()
    endif()

    set(${SRCS})
    set(${HDRS})
    foreach(FIL ${ARGN})
        get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
        get_filename_component(FIL_WE ${FIL} NAME_WE)

        list(APPEND ${SRCS} "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.cc")
        list(APPEND ${HDRS} "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.h")

        add_custom_command(
                OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.cc"
                "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.h"
                COMMAND  ${Protobuf_PROTOC_EXECUTABLE}
                ARGS --grpc_out=${CMAKE_CURRENT_BINARY_DIR}
                --plugin=protoc-gen-grpc=${GRPC_CPP_PLUGIN}
                ${_protobuf_include_path} ${ABS_FIL}
                DEPENDS ${ABS_FIL} ${Protobuf_PROTOC_EXECUTABLE}
                COMMENT "Running gRPC C++ protocol buffer compiler on ${FIL}"
                VERBATIM)
    endforeach()

    set_source_files_properties(${${SRCS}} ${${HDRS}} PROPERTIES GENERATED TRUE)
    set(${SRCS} ${${SRCS}} PARENT_SCOPE)
    set(${HDRS} ${${HDRS}} PARENT_SCOPE)
endfunction()

include_directories(${PROTOBUF_INCLUDE_DIRS})

file(GLOB GRPCProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/rpc.proto")
PROTOBUF_GENERATE_GRPC_CPP(ProtoGRPCSources ProtoGRPCHeaders ${GRPCProtoFiles})
set(ProtoGRPCSources ${ProtoGRPCSources} PARENT_SCOPE)
set(ProtoGRPCHeaders ${ProtoGRPCHeaders} PARENT_SCOPE)

file(GLOB ProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/*.proto")
PROTOBUF_GENERATE_CPP(ProtoSources ProtoHeaders ${ProtoFiles})
set(ProtoSources ${ProtoSources} PARENT_SCOPE)
set(ProtoHeaders ${ProtoHeaders} PARENT_SCOPE)

add_custom_target(proto_generating
        DEPENDS ${ProtoGRPCSources} ${ProtoSources})
