#[[
Copyright (c) 2022 TUM Department of Electrical and Computer Engineering.

This file is part of MLonMCU.
See https://github.com/tum-ei-eda/mlonmcu.git for further info.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
]]
cmake_minimum_required(VERSION 3.5)

SET(TVM_SRC ${CONFIG_TVM_DIR})
SET(TVM_CRT_CONFIG_DIR ${CONFIG_TVM_CRT_CONFIG_DIR})

# If TVM_SRC is not configured, use the bundled runtime
IF ("${TVM_SRC}" STREQUAL "" OR "${TVM_SRC}" STREQUAL "$(TVM_SRC)")
    SET(TVM_SRC ${CMAKE_CURRENT_SOURCE_DIR}/runtime)
ENDIF()

IF (NOT ${TVM_SRC} STREQUAL "")

SET(TVM_ALIGNMENT_BYTES 4)

# cmake-format: off
SET(TVM_HEADERS
    ${TVM_SRC}/include
)

IF(EXISTS "${TVM_SRC}/3rdparty/dlpack/include")
    LIST(APPEND TVM_HEADERS 
        ${TVM_SRC}/3rdparty/dlpack/include
        ${TVM_SRC}/apps/bundle_deploy
        ${TVM_SRC}/src/runtime/crt/include
    )
    SET(TVM_CRT_PATH "${TVM_SRC}/src/runtime/crt")
ELSE()
    SET(TVM_CRT_PATH "${TVM_SRC}/src/runtime/crt")
ENDIF()
# cmake-format: on

# Add crt_config directory
IF(TVM_CRT_CONFIG_DIR AND NOT "${TVM_CRT_CONFIG_DIR}" STREQUAL "")
    LIST(APPEND TVM_HEADERS ${TVM_CRT_CONFIG_DIR})
ELSE()
    # Use bundled crt_config
    LIST(APPEND TVM_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/crt_config)
ENDIF()

    idf_component_register(
            SRCS
                ${TVM_CRT_PATH}/common/crt_backend_api.c
                ${TVM_CRT_PATH}/memory/stack_allocator.c
                ${TVM_CRT_PATH}/common/crt_runtime_api.c
                ${TVM_CRT_PATH}/common/func_registry.c
                ${TVM_CRT_PATH}/memory/page_allocator.c
                ${TVM_CRT_PATH}/common/ndarray.c
                ${TVM_CRT_PATH}/common/packed_func.c
                ${TVM_CRT_PATH}/graph_executor/graph_executor.c
                ${TVM_CRT_PATH}/graph_executor/load_json.c
            INCLUDE_DIRS
                ${TVM_HEADERS}
    )


component_compile_options(-Wno-error=unused-but-set-parameter -Wno-error=format -Wno-incompatible-pointer-types)

TARGET_INCLUDE_DIRECTORIES(${COMPONENT_LIB} PUBLIC
    -DTVM_RUNTIME_ALLOC_ALIGNMENT_BYTES=${TVM_ALIGNMENT_BYTES}
)
ELSE()
    idf_component_register()
ENDIF()
