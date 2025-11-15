#
# Model Component CMakeLists.txt
# Auto-generated TVM model compiled by MLonMCU
#

# Collect all .c files from codegen/host/src directory
file(GLOB MODEL_SRCS "codegen/host/src/*.c")

# Add the MLonMCU-generated TVM wrapper
list(APPEND MODEL_SRCS "aot_wrapper.c")

# Register the model component with ESP-IDF
idf_component_register(
    SRCS 
        ${MODEL_SRCS}
    INCLUDE_DIRS 
        "codegen/host/include"
        "."  # Include current directory for tvm_wrapper.h
    REQUIRES 
        tvm-lib
    PRIV_REQUIRES
        mlif2  # Private dependency for printing.h and exit.h headers
)

