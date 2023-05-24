cmake_minimum_required(VERSION 3.0)
project(PerceptionSystem)

option(COMPILE_SL "Compile SLAM" ON)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall   -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -march=native")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")

# Check C++17 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++2a" COMPILER_SUPPORTS_CXX2A)
if(COMPILER_SUPPORTS_CXX2A)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++2a")
   add_definitions(-DCOMPILEDWITHC2A)
   message(STATUS "Using flag -std=c++2a.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++17 support. Please use a different C++ compiler.")
endif()

find_package(realsense2)

add_subdirectory(${PROJECT_SOURCE_DIR}/libraries)

include_directories(
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/libraries
  ${realsense_INCLUDE_DIR}
)

target_link_libraries(${PROJECT_NAME}
  ${realsense2_LIBRARY}
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)

add_executable(perception_system
        src/rgbd_inertial_realsense_D455.cc)
target_link_libraries(perception_system ${PROJECT_NAME})