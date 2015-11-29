# require cmake 2.8.0 for findCUDA.cmake
CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0 FATAL_ERROR)

##################
project("bicgstab_cuda_tuto")
# The version number.
set (bicgstab_cuda_tuto_VERSION_MAJOR 0)
set (bicgstab_cuda_tuto_VERSION_MINOR 0)
##################

###################################################
# DEBUG or RELEASE ? DEBUG is the default behavior
###################################################
if(NOT CMAKE_BUILD_TYPE)
  message(STATUS "Setting build type to 'Debug' as none was specified.")
  set(CMAKE_BUILD_TYPE "Debug")
endif()

# Set default install path to build
#if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
#  set (CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}" CACHE PATH "default install path" FORCE )
#endif()

######################################################
# allow cmake to use custom modules (e.g. Find???)
######################################################
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

find_package(CUDA REQUIRED)
include(FindCUDA)
# Make the include path visible to all targets
INCLUDE_DIRECTORIES (
  ${bicgstab_cuda_tuto_SOURCE_DIR}/src
  )

set( CMAKE_C_FLAGS "-Wall ${CMAKE_C_FLAGS}" )
set( CMAKE_CXX_FLAGS "-Wall ${CMAKE_CXX_FLAGS}" )

#############
# GPU BUILD
#############
# lineinfo is usefull for debugging
list(APPEND CUDA_NVCC_FLAGS --compiler-options -fno-strict-aliasing -lineinfo)
#list(APPEND CUDA_NVCC_FLAGS --ptxas-options=-v)
#list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_20,code=sm_20)
list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_30,code=sm_30)
#list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_35,code=sm_35)
#list(APPEND CUDA_NVCC_FLAGS -gencode arch=compute_37,code=sm_37)

# bicgstab_cuda
set ( bicgstab_cuda_SRCS
  src/bicgstab.cu
  src/blas.cu
  src/jacobi.cu
  src/matrix.cpp
  src/solver.cu
  src/spmv.cu
  src/utils.cpp
  src/main.cpp
  )
cuda_add_executable ( bicgstab_cuda
  ${bicgstab_cuda_SRCS} 
  )

# make sure config.txt and res are copied into build directory
add_custom_command(TARGET bicgstab_cuda PRE_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_directory
  ${CMAKE_SOURCE_DIR}/res $<TARGET_FILE_DIR:bicgstab_cuda>
  COMMAND ${CMAKE_COMMAND} -E copy
  ${CMAKE_SOURCE_DIR}/config.txt $<TARGET_FILE_DIR:bicgstab_cuda>
  )