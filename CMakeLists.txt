cmake_minimum_required(VERSION 3.0)
project(PerceptionSystem)

option(COMPILE_SL "Compile SLAM" ON)
option(COMPILE_YL "Compile YOLO" ON)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall   -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -march=native")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")

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
find_package(OpenCV REQUIRED)

add_subdirectory(
  ${PROJECT_SOURCE_DIR}/libraries
)

include_directories(
  ${PROJECT_SOURCE_DIR}
  ${PROJECT_SOURCE_DIR}/libraries
  ${realsense_INCLUDE_DIR}
  
  ${LIBRARIES_INCLUDE_DIR}
)

add_library(${PROJECT_NAME} STATIC
  src/rgbd_inertial_realsense_D455
)

target_link_libraries(${PROJECT_NAME}
  ${realsense2_LIBRARY}
  ${OpenCV_LIBS}
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)

target_link_libraries(${PROJECT_NAME}
  ${OpenCV_LIBS}
  ${realsense2_LIBRARY}
  ORB_SLAM3
  YOLO_INTERFACE
)

add_executable(perception_system
        src/rgbd_inertial_realsense_D455.cc)
target_link_libraries(perception_system ${PROJECT_NAME} ${OpenCV_LIBS})
