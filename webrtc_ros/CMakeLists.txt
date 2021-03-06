cmake_minimum_required(VERSION 2.8.12)
project(webrtc_ros)

find_package(catkin REQUIRED COMPONENTS
  cv_bridge
  roscpp
  webrtc
  image_transport
  nodelet
  async_web_server_cpp
)

catkin_package(
  CATKIN_DEPENDS cv_bridge roscpp webrtc image_transport nodelet async_web_server_cpp
)

###########
## Build ##
###########

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

# This library contains code that cannot be compiled with c++11 enabled
# This is due to ABI incompatibilities between the two
# While most of ROS appears to work with the webrtc compiler options,
# this is needed for async_web_server_cpp, which relies on Boost ASIO types
# Boost ASIO cannot be mixed between c++11 and non-c++11 compilation units
add_library(webrtc_ros_server_noncpp11
  src/webrtc_web_server.cpp
)
target_link_libraries(webrtc_ros_server_noncpp11
  ${catkin_LIBRARIES}
)

# This library is built with the same options used to build the webrtc library
add_library(webrtc_ros_server
  src/webrtc_ros_message.cpp
  src/configure_message.cpp
  src/sdp_message.cpp
  src/ice_candidate_message.cpp

  src/ros_log_context.cpp

  src/webrtc_ros_server.cpp
  src/webrtc_client.cpp
  src/ros_video_capturer.cpp
  src/ros_video_renderer.cpp
)
target_link_libraries(webrtc_ros_server
  ${catkin_LIBRARIES}
  ${WEBRTC_LIBRARIES}
  webrtc_ros_server_noncpp11
)
target_compile_definitions(webrtc_ros_server PUBLIC ${WEBRTC_COMPILE_DEFINITIONS})
target_compile_options(webrtc_ros_server PUBLIC ${WEBRTC_COMPILE_OPTIONS})
target_include_directories(webrtc_ros_server PUBLIC ${WEBRTC_INCLUDE_DIRS})
set_target_properties(webrtc_ros_server PROPERTIES LINK_FLAGS "${WEBRTC_LINK_FLAGS}")

add_executable(webrtc_ros_server_node
  src/webrtc_ros_server_node.cpp
)
target_link_libraries(webrtc_ros_server_node
  ${catkin_LIBRARIES}
  webrtc_ros_server
)

add_library(webrtc_ros_server_nodelet
  src/webrtc_ros_server_nodelet.cpp
)
target_link_libraries(webrtc_ros_server_nodelet
  ${catkin_LIBRARIES}
  webrtc_ros_server
)


#############
## Install ##
#############

# Mark nodes and libraries for installation
install(TARGETS
  webrtc_ros_server_noncpp11
  webrtc_ros_server
  webrtc_ros_server_node
  webrtc_ros_server_nodelet

  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

# Mark nodelet plugin configuration for installation
install(FILES
  nodelet_plugins.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

# Mark cpp header files for installation
install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

# Install web files
install(DIRECTORY web
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  PATTERN "Gruntfile.js" EXCLUDE
  PATTERN "node_modules" EXCLUDE
  PATTERN "package.json" EXCLUDE
  PATTERN ".gitignore" EXCLUDE
)

# Install license files
install(FILES
  ../LICENSE
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(FILES
  LICENSE_webrtc_adapter
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
