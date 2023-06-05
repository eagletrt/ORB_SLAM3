cmake_minimum_required(VERSION 3.0)
project(Libs)

add_subdirectory(${PROJECT_SOURCE_DIR}/SLAM)
add_subdirectory(${PROJECT_SOURCE_DIR}/YOLOv8)


set(LIBRARIES_INCLUDE_DIR 
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_SOURCE_DIR}/SLAM
    ${PROJECT_SOURCE_DIR}/YOLOv8

    ${SLAM_INCLUDE_DIR}
    ${YOLO_INCLUDE_DIR}

    PARENT_SCOPE
)

message(STATUS "slam libraries ----> ${SLAM_INCLUDE_DIR}")
message(STATUS "yolo libraries ----> ${YOLO_INCLUDE_DIR}")
