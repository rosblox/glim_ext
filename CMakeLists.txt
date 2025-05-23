cmake_minimum_required(VERSION 3.5)
project(glim_ext)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/cmake")

# Odometry estimation modules
option(ENABLE_ORBSLAM "Enable ORB-SLAM odometry" OFF)
option(ENABLE_VELSUPP "Enable Velocity suppressor" OFF)
option(ENABLE_IMUVAL "Enable IMU calibration validator" OFF)
option(ENABLE_GRAVITY "Enable Gravity estimation and correction module" OFF)

# Local and global mapping modules
option(ENABLE_SCAN_CONTEXT "Enable ScanContext Loop Detector" ON)
option(ENABLE_DBOW "Enable DBoW Loop Detector" OFF)
option(ENABLE_GNSS "Enable GNSS module (ROS1 is not supported)" OFF)
option(ENABLE_FLATEARTHER "Enable flat earth assumption module" OFF)

#################
## ROS-related ##
#################
if(DEFINED ENV{ROS_VERSION})
  if($ENV{ROS_VERSION} EQUAL 2)
    find_package(ament_cmake REQUIRED)
  elseif($ENV{ROS_VERSION} EQUAL 1)
    # ROS1
    find_package(catkin REQUIRED)
  endif()
endif()

find_package(glim REQUIRED)

add_library(glim_ext SHARED
  src/glim_ext/util/config_ext.cpp
)
target_include_directories(glim_ext PUBLIC
  include
)
target_link_libraries(glim_ext
  glim::glim
)
target_compile_options(glim_ext PRIVATE
  -std=c++17
)
list(APPEND glim_ext_LIBRARIES glim_ext)

# Examples
add_subdirectory(modules/example/glim_callback_demo)
list(APPEND glim_ext_LIBRARIES glim_callback_demo)

# Odometry estimation modules
if(ENABLE_ORBSLAM)
  add_subdirectory(modules/odometry/orb_slam_odometry)
  list(APPEND glim_ext_LIBRARIES orb_slam_odometry)
endif()

if(ENABLE_VELSUPP)
  add_subdirectory(modules/odometry/velocity_suppressor)
  list(APPEND glim_ext_LIBRARIES velocity_suppressor)
endif()

if(ENABLE_IMUVAL)
  add_subdirectory(modules/odometry/imu_validator)
  list(APPEND glim_ext_LIBRARIES imu_validator)
endif()

if(ENABLE_GRAVITY)
  add_subdirectory(modules/odometry/gravity_estimator)
  list(APPEND glim_ext_LIBRARIES gravity_estimator)
endif()

# Local and global mapping modules
if(ENABLE_SCAN_CONTEXT)
  add_subdirectory(modules/mapping/scan_context_loop_detector)
  list(APPEND glim_ext_LIBRARIES scan_context_loop_detector)
endif()

if(ENABLE_DBOW)
  add_subdirectory(modules/mapping/dbow_loop_detector)
  list(APPEND glim_ext_LIBRARIES dbow_loop_detector)
endif()

if(ENABLE_GNSS)
  add_subdirectory(modules/mapping/gnss_global)
  list(APPEND glim_ext_LIBRARIES gnss_global)
endif()

if(ENABLE_FLATEARTHER)
  add_subdirectory(modules/mapping/flat_earther)
  list(APPEND glim_ext_LIBRARIES flat_earther)
endif()

#############
## Install ##
#############

if(DEFINED ENV{ROS_VERSION})
  if($ENV{ROS_VERSION} EQUAL 1)
    # No idea
    set(CMAKE_INSTALL_INCLUDEDIR "include/glim")
    set(CMAKE_INSTALL_LIBDIR "lib")
    set(CMAKE_INSTALL_BINDIR "bin/glim")
    set(CMAKE_INSTALL_DATADIR "share/glim")
    set(CMAKE_CONFIG_INSTALL_DIR "share/glim")
  elseif($ENV{ROS_VERSION} EQUAL 2)
    set(CMAKE_INSTALL_INCLUDEDIR "include/glim")
    set(CMAKE_INSTALL_LIBDIR "lib")
    set(CMAKE_INSTALL_BINDIR "bin/glim")
    set(CMAKE_INSTALL_DATADIR "share/glim")
    set(CMAKE_CONFIG_INSTALL_DIR "share/glim")
  else()
    message(FATAL_ERROR "Unsupported ROS version")
  endif()
else()
  include(GNUInstallDirs)
  set(CMAKE_CONFIG_INSTALL_DIR
    "${CMAKE_INSTALL_LIBDIR}/cmake/glim"
    CACHE PATH "Install directory for CMake config files"
  )
endif()

install(TARGETS ${glim_ext_LIBRARIES}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

#################
## ROS-related ##
#################
if(DEFINED ENV{ROS_VERSION})
  if($ENV{ROS_VERSION} EQUAL 2)
    # ROS2
    install(DIRECTORY config DESTINATION share/glim_ext)
    ament_target_dependencies(glim_ext glim)
    ament_export_libraries(${glim_ext_LIBRARIES})
    ament_package()
  elseif($ENV{ROS_VERSION} EQUAL 1)
    # ROS1
    catkin_package(
      LIBRARIES ${glim_ext_LIBRARIES}
    )
  endif()
endif()
