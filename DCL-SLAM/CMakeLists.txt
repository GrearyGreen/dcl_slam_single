cmake_minimum_required(VERSION 2.8.3)
project(dcl_slam)

set(CMAKE_BUILD_TYPE "Release")
# set(CMAKE_CXX_FLAGS "-std=c++11")
set(CMAKE_CXX_STANDARD 14)
#set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g -pthread")

find_package(catkin REQUIRED COMPONENTS
  # distributed_mapper_catkin
  distributed_mapper
  # gtsam_catkin
  # glog_catkin
  tf
  roscpp
  std_msgs
  sensor_msgs
  geometry_msgs
  nav_msgs
  message_generation
  visualization_msgs
)

set(OpenCV_DIR /usr/local/share/opencv4)
find_package(Boost REQUIRED COMPONENTS filesystem system thread)
find_package(Eigen3 REQUIRED)
find_package(libnabo)
find_package(PCL REQUIRED)
find_package(OpenMP REQUIRED)
find_package(OpenCV REQUIRED)
find_package(glog REQUIRED)
find_package(GTSAM REQUIRED)

if(NOT GTSAM_FOUND)
  message(FATAL_ERROR "GTSAM library not found!")
endif()

find_package(Boost REQUIRED COMPONENTS 
  serialization 
  system 
  filesystem 
  thread 
  date_time 
  timer 
  chrono)

add_message_files(
  DIRECTORY msg
  FILES
  loop_info.msg
  global_descriptor.msg
  neighbor_estimate.msg
)

generate_messages(
  DEPENDENCIES
  geometry_msgs
  std_msgs
  nav_msgs
  sensor_msgs
)

catkin_package(
  INCLUDE_DIRS include
  DEPENDS 
  GTSAM
  CATKIN_DEPENDS
  std_msgs
  nav_msgs
  geometry_msgs
  sensor_msgs
  message_runtime
  message_generation
  visualization_msgs
  # distributed_mapper_catkin
  distributed_mapper
  LIBRARIES dcl_slam
)

# include directories
include_directories(
  include
  ${PCL_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${GTSAM_INCLUDE_DIR}
  ${glog_INCLUDE_DIR}
)

# link directories
link_directories(
  include
  ${PCL_LIBRARY_DIRS}
  ${OpenCV_LIBRARY_DIRS}
  ${libnabo_LIBRARY_DIRS}
  ${GTSAM_LIBRARY_DIRS}
)

###########
## Build ##
###########
# dcl slam libraay
add_library(dcl_slam
  src/paramsServer.cpp
  src/distributedMappingBasis.cpp 
  src/distributedMapping.cpp
  src/distributedMappingVisualization.cpp
  src/distributedLoopClosure.cpp
  src/scanContextDescriptor.cpp
  src/lidarIrisDescriptor.cpp
  src/m2dpDescriptor.cpp)
target_link_libraries(dcl_slam ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${OpenCV_LIBRARIES} 
  ${OpenMP_CXX_FLAGS} 
  ${libnabo_LIBRARIES} 
  glog::glog 
  Eigen3::Eigen
  )

add_executable(${PROJECT_NAME}_loopVisualizationNode src/loopVisualizationNode.cpp)
add_dependencies(${PROJECT_NAME}_loopVisualizationNode ${catkin_EXPORTED_TARGETS} ${PROJECT_NAME}_generate_messages_cpp)
target_compile_options(${PROJECT_NAME}_loopVisualizationNode PRIVATE ${OpenMP_CXX_FLAGS})
target_link_libraries(${PROJECT_NAME}_loopVisualizationNode ${catkin_LIBRARIES} ${PCL_LIBRARIES} 
  ${OpenCV_LIBRARIES} ${OpenMP_CXX_FLAGS} 
  ${libnabo_LIBRARIES} 
  gtsam
  boost_serialization
  boost_system
  boost_filesystem
  boost_thread
  boost_date_time
  boost_timer
  boost_chrono
  glog::glog 
  Eigen3::Eigen
  )
