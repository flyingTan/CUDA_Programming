cmake_minimum_required(VERSION 2.8.4)

### Compiler settings
set( CMAKE_CXX_FLAGS_DEBUG "-g -O0" )
set( CMAKE_CXX_FLAGS_RELEASE "-O3" )
set( CMAKE_CXX_STANDARD_REQUIRED ON ) #...is required...
set( CMAKE_CXX_EXTENSIONS ON ) #...with compiler extensions like gnu++11

set( CMAKE_VERBOSE_MAKEFILE ON )

## Include header
include_directories(
    ${PROJECT_SOURCE_DIR}/include
    "/usr/local/cuda-10.0/include"
    )

## Add cuda library
set( CMAKE_CUDA_COMPILER "/usr/local/cuda/bin/nvcc")
find_package(CUDA REQUIRED)
if(NOT CUDA_FOUND)
    message(FATAL_ERROR "CUDA not found")
else()
    message(STATUS "LOG: FIND CUDA")
    enable_language("CUDA")
endif()

find_library(ADD_LIB NAMES add PATHS ${PROJECT_BINARY_DIR}/lib)

message(STATUS "Library path ADD_LIB is: " ${ADD_LIB})

## Sample Target
file(GLOB CPP_FILES *.cpp)
set(CUDA_SEPARABLE_COMPILATION ON)
include_directories(/usr/local/cuda/include)
add_definitions(-DUSE_GPU)
foreach(SAMPLE_SOURCE ${CPP_FILES})
    string(REGEX REPLACE ".+/(.+)\\..*" "\\1" SAMPLE_NAME ${SAMPLE_SOURCE})
    message(STATUS "LOGO: ${SAMPLE_NAME} ${SAMPLE_SOURCE}")
    add_executable(${SAMPLE_NAME} ${SAMPLE_SOURCE})
    target_link_libraries(${SAMPLE_NAME} ${CUDA_LIBRARIES})
    target_link_libraries(${SAMPLE_NAME} ${ADD_LIB})
    target_link_libraries(${SAMPLE_NAME} rt)
    set_target_properties(${SAMPLE_NAME} PROPERTIES CUDA_SEPARABLE_COMPILATION ON)
    link_directories(/usr/local/cuda/lib64)
    ## Install
    install( TARGETS ${SAMPLE_NAME} RUNTIME DESTINATION bin)
endforeach(SAMPLE_SOURCE)