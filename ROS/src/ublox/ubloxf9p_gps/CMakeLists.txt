cmake_minimum_required(VERSION 2.8.3)
project(ubloxf9p_gps)
find_package(catkin REQUIRED COMPONENTS
  roscpp
  roscpp_serialization
  ubloxf9p_msgs
  ubloxf9p_serialization
  diagnostic_updater
)

catkin_package(
    INCLUDE_DIRS include
    LIBRARIES ${PROJECT_NAME}
    CATKIN_DEPENDS roscpp ubloxf9p_msgs ubloxf9p_serialization)

# include boost
find_package(Boost REQUIRED COMPONENTS system regex thread)
link_directories(${Boost_LIBRARY_DIR})
include_directories(${Boost_INCLUDE_DIR})

# include other ublox packages
include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${catkin_INCLUDE_DIRS})

# link pthread
SET(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -std=c++11 -pthread")

# build library
add_library(ubloxf9p_gps src/gps.cpp)

# fix msg compile order bug
add_dependencies(ubloxf9p_gps ${catkin_EXPORTED_TARGETS})

target_link_libraries(ubloxf9p_gps
  boost_system
  boost_regex
  boost_thread
)

target_link_libraries(ubloxf9p_gps
  ${catkin_LIBRARIES}
)

# build node
add_executable(ublox_gps_node src/node.cpp src/mkgmtime.c)
set_target_properties(ublox_gps_node PROPERTIES OUTPUT_NAME ubloxf9p_gps)

target_link_libraries(ublox_gps_node boost_system boost_regex boost_thread)
target_link_libraries(ublox_gps_node ${catkin_LIBRARIES})
target_link_libraries(ublox_gps_node ubloxf9p_gps)

install(TARGETS ubloxf9p_gps ublox_gps_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(PROGRAMS
   scripts/ublox_gps_common_generator.py
   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
 )


install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY config/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/config
  PATTERN ".svn" EXCLUDE
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
  PATTERN ".svn" EXCLUDE
)
