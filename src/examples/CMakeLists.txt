cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

set(PROJECT_VERSION 1.0.1)

project(intrinsic_calib)

#####
set( CMAKE_CXX_FLAGS "-std=c++11")

# if(CAMODOCAL_CALIB_FOUND AND OpenCV_FOUND AND HAVE_OPENCV_XFEATURES2D_NONFREE)
# Set the search paths for include files
include_directories(
  # External
  ${EIGEN3_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS}

  # All library includes are prefixed with the path to avoid conflicts
  ${CMAKE_CURRENT_SOURCE_DIR}

  # testing
  ${GTEST_INCLUDE_DIRS}

  # CamOdoCalConfig.h
  ${CMAKE_CURRENT_BINARY_DIR}
)
# Global includes
include_directories(
  ${PROJECT_SOURCE_DIR}/../../include
  # include/camodocal/*
  ${PROJECT_SOURCE_DIR}/../
  ${PROJECT_SOURCE_DIR}/../chessboard
)

# GTest
find_package(GTest REQUIRED)
include_directories(${GTest_INCLUDE_DIRS})

# boost
message("test")
find_package(Boost  REQUIRED COMPONENTS system filesystem thread)
if(Boost_FOUND)
  include_directories(${Boost_INCLUDE_DIRS})
  link_directories(${Boost_LIBRARY_DIRS})
  add_definitions(-DBOOST_ALL_DYN_LINK)
endif()
# include_directories(${Boost_INCLUDE_DIRS})

message("Boost_INCLUDE_DIRS: ${Boost_INCLUDE_DIRS}")
message("Boost_VERSION: ${Boost_VERSION}")
# set(CMAKE_CXX_FLAGS "-lboost_system -lboost_thread -lboost_program_options")
# add_definitions(${CMAKE_CXX_FLAGS})


# OpenCV
find_package( OpenCV 3 REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )

# Eigen
include_directories( "/usr/include/eigen3" )

# ceres
find_package( Ceres REQUIRED )
include_directories( ${CERES_INCLUDE_DIRS} )

# camodocal_executable(intrinsic_calib
#   intrinsic_calib.cc
# )

message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")

# add_library(ceres SHARED
# )

# add_library(camodocal_gpl SHARED
#   # ${PROJECT_SOURCE_DIR}/../../src/gpl/CubicSpline.cc
#   ${PROJECT_SOURCE_DIR}/../../src/gpl/EigenQuaternionParameterization.cc
#   ${PROJECT_SOURCE_DIR}/../../src/gpl/gpl.cc
# )
SET(GPL_SRC_FILES
  # CubicSpline.cc
  ${PROJECT_SOURCE_DIR}/../../src/gpl/EigenQuaternionParameterization.cc
  ${PROJECT_SOURCE_DIR}/../../src/gpl/gpl.cc
)
add_library(camodocal_gpl SHARED ${GPL_SRC_FILES})
target_link_libraries(camodocal_gpl
  -lboost_system -lboost_thread -lboost_program_options
  ${Boost_LIBRARIES}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_THREAD_LIBRARY}
  ${OpenCV_LIBS}
  # ceres
  ${CERES_LIBRARIES}
)

add_library(camodocal_camera_models SHARED
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/Camera.cc
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/CameraFactory.cc
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/PinholeCamera.cc
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/ScaramuzzaCamera.cc
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/EquidistantCamera.cc
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/CostFunctionFactory.cc
  ${PROJECT_SOURCE_DIR}/../../src/camera_models/CataCamera.cc
)
target_link_libraries(camodocal_camera_models
  # ${OpenCV_LIBS}
  -lboost_system -lboost_thread -lboost_program_options
  ${PROJECT_SOURCE_DIR}/../../src/gpl/gpl.cc
  ${Boost_LIBRARIES}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_THREAD_LIBRARY}
  camodocal_gpl
  # ${CERES_LIBRARIES}
  ${GLOG_LIBRARIES}
  ${GTEST_LIBRARIES}
)

add_library(camodocal_sparse_graph SHARED
  ${PROJECT_SOURCE_DIR}/../../src/sparse_graph/Transform.cc
  ${PROJECT_SOURCE_DIR}/../../src/sparse_graph/Odometry.cc
  ${PROJECT_SOURCE_DIR}/../../src/sparse_graph/Pose.cc
)
target_link_libraries(camodocal_sparse_graph
  -lboost_system -lboost_thread -lboost_program_options
  ${Boost_LIBRARIES}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_THREAD_LIBRARY}
  ${OpenCV_LIBS}
)

# camodocal_calib
add_library(camodocal_calib SHARED
  ${PROJECT_SOURCE_DIR}/../../src/calib/CameraCalibration.cc
  ${PROJECT_SOURCE_DIR}/../../src/calib/utils.cc
)
target_link_libraries(camodocal_calib
  -lboost_system -lboost_thread -lboost_program_options
  ${Boost_LIBRARIES}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_THREAD_LIBRARY}
  # ${CALIB_OPTIONAL_DEPENDENCIES}
  # ${OpenCV_LIBS}
  camodocal_camera_models
  # camodocal_camera_systems
  camodocal_sparse_graph
  camodocal_gpl
  # ${CERES_LIBRARIES}
)

# link camodocal_chessboard
add_library(camodocal_chessboard SHARED
  ${PROJECT_SOURCE_DIR}/../../src/chessboard/Chessboard.cc
)
target_link_libraries(camodocal_chessboard

  # ${CAMODOCAL_PLATFORM_UNIX_LIBRARIES}
  ${OpenCV_LIBS}
)

add_executable(intrinsic_calib
  intrinsic_calib.cc)

# camodocal_link_libraries(intrinsic_calib
#   ${CAMODOCAL_PLATFORM_UNIX_LIBRARIES}
#   ${Boost_FILESYSTEM_LIBRARY}
#   ${Boost_PROGRAM_OPTIONS_LIBRARY}
#   ${Boost_SYSTEM_LIBRARY}
#   ${OpenCV_LIBS}
#   camodocal_calib
#   camodocal_chessboard
# )
target_link_libraries(intrinsic_calib
  -lboost_system -lboost_thread -lboost_program_options
  ${Boost_LIBRARIES}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SYSTEM_LIBRARY}
  ${Boost_THREAD_LIBRARY}
  # ${OpenCV_LIBS}
  camodocal_calib
  camodocal_chessboard
  camodocal_gpl
)

###

add_executable(test
  test.cc)

# camodocal_executable(stereo_calib
#   stereo_calib.cc
# )

# camodocal_link_libraries(stereo_calib
#   ${CAMODOCAL_PLATFORM_UNIX_LIBRARIES}
#   ${Boost_FILESYSTEM_LIBRARY}
#   ${Boost_PROGRAM_OPTIONS_LIBRARY}
#   ${Boost_SYSTEM_LIBRARY}
#   ${OpenCV_LIBS}
#   camodocal_calib
#   camodocal_chessboard
# )

# camodocal_executable(extrinsic_calib
#   extrinsic_calib.cc
# )

# camodocal_link_libraries(extrinsic_calib
#   ${CAMODOCAL_PLATFORM_UNIX_LIBRARIES}
#   ${Boost_FILESYSTEM_LIBRARY}
#   ${Boost_PROGRAM_OPTIONS_LIBRARY}
#   ${Boost_SYSTEM_LIBRARY}
#   camodocal_calib
# )

# endif(CAMODOCAL_CALIB_FOUND AND OpenCV_FOUND AND HAVE_OPENCV_XFEATURES2D_NONFREE)

# if(OpenCV_FOUND)
# include_directories(
#   ../dbow2/DBoW2
#   ../dbow2/DUtils
#   ../dbow2/DUtils
#   ../dbow2/DUtilsCV
#   ../dbow2/DVision
# )

# camodocal_executable(train_voctree
#   train_voctree.cc
# )

# camodocal_link_libraries(train_voctree
#   camodocal_dbow2
# )
# endif()
