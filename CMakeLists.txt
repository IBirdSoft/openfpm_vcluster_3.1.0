cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(openfpm_pdata LANGUAGES C CXX)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake_modules/)

if (POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

set(BOOST_INCLUDE ${Boost_INCLUDE_DIR} CACHE PATH "Include directory for BOOST")
set(SE_CLASS1 CACHE BOOL "Activate compilation with SE_CLASS1")
set(SE_CLASS2 CACHE BOOL "Activate compilation with SE_CLASS2")
set(SE_CLASS3 CACHE BOOL "Activate compilation with SE_CLASS3")
set(PETSC_ROOT CACHE PATH "If compiling with linear algebra indicate the PETSC root directory")
set(ALPAKA_ROOT CACHE PATH "Alpaka root path")
set(CUDA_ON_CPU CACHE BOOL "Make Cuda work on heap")

execute_process(COMMAND bash -c "cp ${CMAKE_CURRENT_SOURCE_DIR}/src/VCluster/VCluster.cpp ${CMAKE_CURRENT_SOURCE_DIR}/src/VCluster/VCluster_cuda_on_cpu.cpp")

if (ENABLE_GPU)
    set(CUDA_ON_CPU OFF)
    enable_language(CUDA)
    find_package(CUDA)
endif()

set(ENV{PETSC_DIR} ${PETSC_ROOT})

set (CMAKE_CXX_STANDARD 14)
set (CMAKE_CUDA_STANDARD 14)

find_package(Boost 1.72.0 REQUIRED COMPONENTS unit_test_framework iostreams program_options OPTIONAL_COMPONENTS fiber context)
find_package(MPI REQUIRED)
find_package(PETSc)

if(CUDA_FOUND)
	set(OPENFPM_INIT_FILE "initialize/initialize_wrapper_cuda.cu")
else()
	set(OPENFPM_INIT_FILE "initialize/initialize_wrapper_cuda.cpp")
endif()

###### CONFIG.h FILE ######

if(SE_CLASS1)
	set(DEFINE_SE_CLASS1 "#define SE_CLASS1")
endif()

if(SE_CLASS2)
	set(DEFINE_SE_CLASS2 "#define SE_CLASS2")
endif()

if(SE_CLASS3)
	set(DEFINE_SE_CLASS3 "#define SE_CLASS3")
endif()

if(PETSC_FOUND)
	set(DEFINE_HAVE_PETSC "#define HAVE_PETSC")
endif()

if(MPI_FOUND)
	set(DEFINE_HAVE_MPI "#define HAVE_MPI")
else()
	message( FATAL_ERROR "MPI is required in order to install OpenFPM" )
endif()

if (Boost_FOUND)
        set(DEFINE_HAVE_BOOST "#define HAVE_BOOST")
        set(DEFINE_HAVE_BOOST_IOSTREAMS "#define HAVE_BOOST_IOSTREAMS")
        set(DEFINE_HAVE_BOOST_PROGRAM_OPTIONS "#define HAVE_BOOST_PROGRAM_OPTIONS")
        set(DEFINE_HAVE_BOOST_UNIT_TEST_FRAMEWORK "#define HAVE_BOOST_UNIT_TEST_FRAMEWORK")
        if (Boost_CONTEXT_FOUND)
                set(DEFINE_HAVE_BOOST_CONTEXT "#define HAVE_BOOST_CONTEXT")
        else()
                #if context is not there CUDA_ON_CPU cannot be activated
                set(CUDA_ON_CPU OFF)
        endif()
        if (Boost_FIBER_FOUND)
                set(DEFINE_HAVE_BOOST_FIBER "#define HAVE_BOOST_FIBER")
        endif()
else()
	message( FATAL_ERROR "BOOST is required in order to install OpenFPM" )
endif()

if(CUDA_FOUND)
        set(DEFINE_CUDA_GPU "#define CUDA_GPU")
endif()

if(CUDA_ON_CPU)
        set(DEFINE_CUDA_GPU "#define CUDA_GPU")
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/config/config_cmake.h.in ${CMAKE_CURRENT_SOURCE_DIR}/src/config/config.h)

include_directories(SYSTEM ${MPI_INCLUDE_PATH})

add_subdirectory (../openfpm_data/ openfpm_data)

set(WARNING_SUPPRESSION_AND_OPTION_NVCC ${WARNING_SUPPRESSION_AND_OPTION_NVCC} PARENT_SCOPE)
set(WARNING_SUPPRESSION_AND_OPTION_NVCC_TEXT ${WARNING_SUPPRESSION_AND_OPTION_NVCC_TEXT} PARENT_SCOPE)

add_subdirectory (src)
