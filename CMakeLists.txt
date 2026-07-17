cmake_minimum_required(VERSION 3.21)
project(docs_embedder LANGUAGES C)

# Ensure C23 standard is used
set(CMAKE_C_STANDARD 23)
set(CMAKE_C_STANDARD_REQUIRED OFF)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# 1. Dependency Discovery
# Guard against redundant and potentially broken lookups in parent builds.
# We always find the Interpreter alongside the Development.Module component
# to ensure CMake resolves the correct virtual environment path on Windows CI.
if(NOT TARGET Python::Module)
    find_package(Python REQUIRED COMPONENTS Interpreter Development.Module QUIET)
endif()

# Define the static library
add_library(docs_embedder STATIC
    src/docs-embedder/docs_embedder.c
)

if(NOT MSVC)
    target_compile_features(docs_embedder PRIVATE c_std_23)
else()
    target_compile_options(docs_embedder PRIVATE $<$<C_COMPILER_ID:Clang>:/clang:-std=c23>)
endif()

# Link against Python headers (Namespace changed to 'Python' to match parent)
# Use 'PRIVATE' for the link, but 'INTERFACE' for the include so parent projects get the headers
target_link_libraries(docs_embedder PRIVATE Python::Module)

# Export the include directory so the parent project can #include "docs_embedder.h"
target_include_directories(docs_embedder
    PUBLIC
        # Path for the library itself and parent projects
        ${CMAKE_CURRENT_SOURCE_DIR}/include
    PRIVATE
        # Path for internal headers only used by the .c files
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# Optional: Add compiler-specific flags for C23 if needed
if(CMAKE_C_COMPILER_ID STREQUAL "Clang" OR CMAKE_C_COMPILER_ID STREQUAL "GNU")
    target_compile_options(docs_embedder PRIVATE -Wall -Wextra -Wpedantic)
elseif(MSVC)
    target_compile_options(docs_embedder PRIVATE /W4)
endif()
