cmake_minimum_required(VERSION 3.13)
project(syz-codesearch)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find LLVM and Clang
# You can point to your LLVM build/install using -DCMAKE_PREFIX_PATH or -DLLVM_DIR/-DClang_DIR
find_package(LLVM REQUIRED CONFIG)
find_package(Clang REQUIRED CONFIG)

message(STATUS "Found LLVM ${LLVM_VERSION}")
message(STATUS "Found Clang ${CLANG_VERSION}")

list(APPEND CMAKE_MODULE_PATH "${LLVM_CMAKE_DIR}")
include(AddLLVM)

# Include paths for LLVM/Clang headers
include_directories(${LLVM_INCLUDE_DIRS} ${CLANG_INCLUDE_DIRS})
add_definitions(${LLVM_DEFINITIONS})

add_executable(syz-codesearch
    codesearch.cpp
)

# Include path for sibling headers (e.g., json.h in ../)
target_include_directories(syz-codesearch PRIVATE ..)

# Link against Clang and LLVM libraries
target_link_libraries(syz-codesearch
    PRIVATE
    clangTooling
    clangFrontend
    clangAST
    clangBasic
    clangSerialization
    LLVMSupport
)

# Apply specific flags mentioned in the original manual build
target_compile_options(syz-codesearch PRIVATE 
    -Wno-c++20-designator 
    -Wno-missing-designated-field-initializers
)

# Ensure configuration matches LLVM (RTTI, etc.)
if(NOT LLVM_ENABLE_RTTI)
    target_compile_options(syz-codesearch PRIVATE -fno-rtti)
endif()
