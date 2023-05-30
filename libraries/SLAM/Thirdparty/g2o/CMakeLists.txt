CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
SET(CMAKE_LEGACY_CYGWIN_WIN32 0)

PROJECT(g2o)

SET(g2o_C_FLAGS)
SET(g2o_CXX_FLAGS)

# default built type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

MESSAGE(STATUS "BUILD TYPE:" ${CMAKE_BUILD_TYPE})

SET (G2O_LIB_TYPE SHARED)

# There seems to be an issue with MSVC8
# see http://eigen.tuxfamily.org/bz/show_bug.cgi?id=83
if(MSVC90)
  add_definitions(-DEIGEN_DONT_ALIGN_STATICALLY=1)
  message(STATUS "Disabling memory alignment for MSVC8")
endif(MSVC90)

# Set the output directory for the build executables and libraries
IF(WIN32)
  SET(g2o_LIBRARY_OUTPUT_DIRECTORY ${g2o_SOURCE_DIR}/bin CACHE PATH "Target for the libraries")
ELSE(WIN32)
  SET(g2o_LIBRARY_OUTPUT_DIRECTORY ${g2o_SOURCE_DIR}/lib CACHE PATH "Target for the libraries")
ENDIF(WIN32)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${g2o_LIBRARY_OUTPUT_DIRECTORY})
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${g2o_LIBRARY_OUTPUT_DIRECTORY})
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${g2o_RUNTIME_OUTPUT_DIRECTORY})

# Set search directory for looking for our custom CMake scripts to
# look for SuiteSparse, QGLViewer, and Eigen3.
LIST(APPEND CMAKE_MODULE_PATH ${g2o_SOURCE_DIR}/cmake_modules)

# Detect OS and define macros appropriately
IF(UNIX)
  ADD_DEFINITIONS(-DUNIX)
  MESSAGE(STATUS "Compiling on Unix")
ENDIF(UNIX)

# Eigen library parallelise itself, though, presumably due to performance issues
# OPENMP is experimental. We experienced some slowdown with it
FIND_PACKAGE(OpenMP)
SET(G2O_USE_OPENMP OFF CACHE BOOL "Build g2o with OpenMP support (EXPERIMENTAL)")
IF(OPENMP_FOUND AND G2O_USE_OPENMP)
  SET (G2O_OPENMP 1)
  SET(g2o_C_FLAGS "${g2o_C_FLAGS} ${OpenMP_C_FLAGS}")
  SET(g2o_CXX_FLAGS "${g2o_CXX_FLAGS} -DEIGEN_DONT_PARALLELIZE ${OpenMP_CXX_FLAGS}")
  MESSAGE(STATUS "Compiling with OpenMP support")
ENDIF(OPENMP_FOUND AND G2O_USE_OPENMP)

# Compiler specific options for gcc
SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -march=native")
SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3 -march=native")
# SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3")
# SET(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3")

# activate warnings !!!
SET(g2o_C_FLAGS "${g2o_C_FLAGS} -Wall -W")
SET(g2o_CXX_FLAGS "${g2o_CXX_FLAGS} -Wall -W")

# specifying compiler flags
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${g2o_CXX_FLAGS}")
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${g2o_C_FLAGS}")

# Find Eigen3
SET(EIGEN3_INCLUDE_DIR ${G2O_EIGEN3_INCLUDE})
FIND_PACKAGE(Eigen3 3.1.0 REQUIRED)
IF(EIGEN3_FOUND)
  SET(G2O_EIGEN3_INCLUDE ${EIGEN3_INCLUDE_DIR} CACHE PATH "Directory of Eigen3")
ELSE(EIGEN3_FOUND)
  SET(G2O_EIGEN3_INCLUDE "" CACHE PATH "Directory of Eigen3")
ENDIF(EIGEN3_FOUND)

# Generate config.h
SET(G2O_CXX_COMPILER "${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER}")
configure_file(config.h.in ${g2o_SOURCE_DIR}/config.h)

# Set up the top-level include directories
INCLUDE_DIRECTORIES(
${g2o_SOURCE_DIR}/core
${g2o_SOURCE_DIR}/types
${g2o_SOURCE_DIR}/stuff 
${G2O_EIGEN3_INCLUDE})

# Include the subdirectories
ADD_LIBRARY(g2o ${G2O_LIB_TYPE}
#types
  ${PROJECT_SOURCE_DIR}/g2o/types/types_sba.h
  ${PROJECT_SOURCE_DIR}/g2o/types/types_six_dof_expmap.h
  ${PROJECT_SOURCE_DIR}/g2o/types/types_sba.cpp
  ${PROJECT_SOURCE_DIR}/g2o/types/types_six_dof_expmap.cpp
  ${PROJECT_SOURCE_DIR}/g2o/types/types_seven_dof_expmap.cpp
  ${PROJECT_SOURCE_DIR}/g2o/types/types_seven_dof_expmap.h
  ${PROJECT_SOURCE_DIR}/g2o/types/se3quat.h
  ${PROJECT_SOURCE_DIR}/g2o/types/se3_ops.h
  ${PROJECT_SOURCE_DIR}/g2o/types/se3_ops.hpp
#core
  ${PROJECT_SOURCE_DIR}/g2o/core/base_edge.h
  ${PROJECT_SOURCE_DIR}/g2o/core/base_binary_edge.h
  ${PROJECT_SOURCE_DIR}/g2o/core/hyper_graph_action.cpp
  ${PROJECT_SOURCE_DIR}/g2o/core/base_binary_edge.hpp
  ${PROJECT_SOURCE_DIR}/g2o/core/hyper_graph_action.h
  ${PROJECT_SOURCE_DIR}/g2o/core/base_multi_edge.h           
  ${PROJECT_SOURCE_DIR}/g2o/core/hyper_graph.cpp
  ${PROJECT_SOURCE_DIR}/g2o/core/base_multi_edge.hpp         
  ${PROJECT_SOURCE_DIR}/g2o/core/hyper_graph.h
  ${PROJECT_SOURCE_DIR}/g2o/core/base_unary_edge.h          
  ${PROJECT_SOURCE_DIR}/g2o/core/linear_solver.h
  ${PROJECT_SOURCE_DIR}/g2o/core/base_unary_edge.hpp         
  ${PROJECT_SOURCE_DIR}/g2o/core/marginal_covariance_cholesky.cpp
  ${PROJECT_SOURCE_DIR}/g2o/core/base_vertex.h               
  ${PROJECT_SOURCE_DIR}/g2o/core/marginal_covariance_cholesky.h
  ${PROJECT_SOURCE_DIR}/g2o/core/base_vertex.hpp             
  ${PROJECT_SOURCE_DIR}/g2o/core/matrix_structure.cpp
  ${PROJECT_SOURCE_DIR}/g2o/core/batch_stats.cpp             
  ${PROJECT_SOURCE_DIR}/g2o/core/matrix_structure.h
  ${PROJECT_SOURCE_DIR}/g2o/core/batch_stats.h               
  ${PROJECT_SOURCE_DIR}/g2o/core/openmp_mutex.h
  ${PROJECT_SOURCE_DIR}/g2o/core/block_solver.h              
  ${PROJECT_SOURCE_DIR}/g2o/core/block_solver.hpp            
  ${PROJECT_SOURCE_DIR}/g2o/core/parameter.cpp               
  ${PROJECT_SOURCE_DIR}/g2o/core/parameter.h                 
  ${PROJECT_SOURCE_DIR}/g2o/core/cache.cpp                   
  ${PROJECT_SOURCE_DIR}/g2o/core/cache.h
  ${PROJECT_SOURCE_DIR}/g2o/core/optimizable_graph.cpp       
  ${PROJECT_SOURCE_DIR}/g2o/core/optimizable_graph.h         
  ${PROJECT_SOURCE_DIR}/g2o/core/solver.cpp                  
  ${PROJECT_SOURCE_DIR}/g2o/core/solver.h
  ${PROJECT_SOURCE_DIR}/g2o/core/creators.h                 
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_factory.cpp
  ${PROJECT_SOURCE_DIR}/g2o/core/estimate_propagator.cpp     
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_factory.h
  ${PROJECT_SOURCE_DIR}/g2o/core/estimate_propagator.h       
  ${PROJECT_SOURCE_DIR}/g2o/core/factory.cpp                 
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_property.h
  ${PROJECT_SOURCE_DIR}/g2o/core/factory.h                   
  ${PROJECT_SOURCE_DIR}/g2o/core/sparse_block_matrix.h
  ${PROJECT_SOURCE_DIR}/g2o/core/sparse_optimizer.cpp  
  ${PROJECT_SOURCE_DIR}/g2o/core/sparse_block_matrix.hpp
  ${PROJECT_SOURCE_DIR}/g2o/core/sparse_optimizer.h
  ${PROJECT_SOURCE_DIR}/g2o/core/hyper_dijkstra.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/hyper_dijkstra.h
  ${PROJECT_SOURCE_DIR}/g2o/core/parameter_container.cpp     
  ${PROJECT_SOURCE_DIR}/g2o/core/parameter_container.h
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm.h
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_with_hessian.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_with_hessian.h
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_levenberg.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_levenberg.h
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_gauss_newton.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/optimization_algorithm_gauss_newton.h
  ${PROJECT_SOURCE_DIR}/g2o/core/jacobian_workspace.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/jacobian_workspace.h
  ${PROJECT_SOURCE_DIR}/g2o/core/robust_kernel.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/robust_kernel.h
  ${PROJECT_SOURCE_DIR}/g2o/core/robust_kernel_factory.cpp
  ${PROJECT_SOURCE_DIR}/g2o/core/robust_kernel_factory.h
  ${PROJECT_SOURCE_DIR}/g2o/core/robust_kernel_impl.cpp 
  ${PROJECT_SOURCE_DIR}/g2o/core/robust_kernel_impl.h
#stuff
  ${PROJECT_SOURCE_DIR}/g2o/stuff/string_tools.h
  ${PROJECT_SOURCE_DIR}/g2o/stuff/color_macros.h 
  ${PROJECT_SOURCE_DIR}/g2o/stuff/macros.h
  ${PROJECT_SOURCE_DIR}/g2o/stuff/timeutil.cpp
  ${PROJECT_SOURCE_DIR}/g2o/stuff/misc.h
  ${PROJECT_SOURCE_DIR}/g2o/stuff/timeutil.h
  ${PROJECT_SOURCE_DIR}/g2o/stuff/os_specific.c    
  ${PROJECT_SOURCE_DIR}/g2o/stuff/os_specific.h
  ${PROJECT_SOURCE_DIR}/g2o/stuff/string_tools.cpp
  ${PROJECT_SOURCE_DIR}/g2o/stuff/property.cpp       
  ${PROJECT_SOURCE_DIR}/g2o/stuff/property.h       
)
