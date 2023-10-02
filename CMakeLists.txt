cmake_minimum_required(VERSION 3.0)
project(ORB_SLAM3)

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

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(OpenCV 4.4)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 4.4 not found.")
   endif()

MESSAGE("OPENCV VERSION:")
MESSAGE(${OpenCV_VERSION})

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/Pangolin)
add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2)
add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/g2o)
add_subdirectory(${PROJECT_SOURCE_DIR}/Thirdparty/Sophus)

set(SLAM_INCLUDE_DIR_INTERNAL

   ${PROJECT_SOURCE_DIR}
   ${PROJECT_SOURCE_DIR}/include
   ${PROJECT_SOURCE_DIR}/include/CameraModels
   ${PROJECT_SOURCE_DIR}/Thirdparty/Sophus
   ${PROJECT_SOURCE_DIR}/Thirdparty/DBoW2
   ${PROJECT_SOURCE_DIR}/Thirdparty/g2o
   ${EIGEN3_INCLUDE_DIR}
   ${Pangolin_INCLUDE_DIRS}
)
set(SLAM_INCLUDE_DIR ${SLAM_INCLUDE_DIR_INTERNAL} PARENT_SCOPE)

include_directories(
   ${SLAM_INCLUDE_DIR_INTERNAL}
)

message(STATUS "SLAM ---- > ${SLAM_INCLUDE_DIR_INTERNAL}")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

add_library(${PROJECT_NAME} SHARED
   ${PROJECT_SOURCE_DIR}/src/System.cc
   ${PROJECT_SOURCE_DIR}/src/Tracking.cc
   ${PROJECT_SOURCE_DIR}/src/LocalMapping.cc
   ${PROJECT_SOURCE_DIR}/src/LoopClosing.cc
   ${PROJECT_SOURCE_DIR}/src/ORBextractor.cc
   ${PROJECT_SOURCE_DIR}/src/ORBmatcher.cc
   ${PROJECT_SOURCE_DIR}/src/FrameDrawer.cc
   ${PROJECT_SOURCE_DIR}/src/Converter.cc
   ${PROJECT_SOURCE_DIR}/src/MapPoint.cc
   ${PROJECT_SOURCE_DIR}/src/KeyFrame.cc
   ${PROJECT_SOURCE_DIR}/src/Atlas.cc
   ${PROJECT_SOURCE_DIR}/src/Map.cc
   ${PROJECT_SOURCE_DIR}/src/MapDrawer.cc
   ${PROJECT_SOURCE_DIR}/src/Optimizer.cc
   ${PROJECT_SOURCE_DIR}/src/Frame.cc
   ${PROJECT_SOURCE_DIR}/src/KeyFrameDatabase.cc
   ${PROJECT_SOURCE_DIR}/src/Sim3Solver.cc
   ${PROJECT_SOURCE_DIR}/src/Viewer.cc
   ${PROJECT_SOURCE_DIR}/src/ImuTypes.cc
   ${PROJECT_SOURCE_DIR}/src/G2oTypes.cc
   ${PROJECT_SOURCE_DIR}/src/CameraModels/Pinhole.cpp
   ${PROJECT_SOURCE_DIR}/src/CameraModels/KannalaBrandt8.cpp
   ${PROJECT_SOURCE_DIR}/src/OptimizableTypes.cpp
   ${PROJECT_SOURCE_DIR}/src/MLPnPsolver.cpp
   ${PROJECT_SOURCE_DIR}/src/GeometricTools.cc
   ${PROJECT_SOURCE_DIR}/src/TwoViewReconstruction.cc
   ${PROJECT_SOURCE_DIR}/src/Config.cc
   ${PROJECT_SOURCE_DIR}/src/Settings.cc
)

target_link_libraries(${PROJECT_NAME}
   ${OpenCV_LIBS}
   ${EIGEN3_LIBS}
   ${Pangolin_LIBRARIES}
   g2o
   DBoW2
   sophus
   -lboost_serialization
   -lcrypto
   -ldl
)