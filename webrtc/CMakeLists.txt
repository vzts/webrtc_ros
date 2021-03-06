cmake_minimum_required(VERSION 2.8.12)
project(webrtc)

find_package(catkin REQUIRED)

set(webrtc_src_root ${PROJECT_SOURCE_DIR}/src)
if(CMAKE_BUILD_TYPE MATCHES Debug)
  set(webrtc_build_root ${CMAKE_BINARY_DIR}/out/Debug)
else()
  set(webrtc_build_root ${CMAKE_BINARY_DIR}/out/Release)
endif()

# checkout source to build and generate ninja build files
execute_process(
  COMMAND ./checkout_source.sh ${CMAKE_BINARY_DIR}
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
  RESULT_VARIABLE CHECKOUT_SOURCE_RESULT
)
if(NOT ${CHECKOUT_SOURCE_RESULT} EQUAL 0)
  message(FATAL_ERROR "WebRTC source checkout failed")
endif()


# generate package cfg extras to be inherited
set(CMAKE_EXTRAS_FILE ${CMAKE_BINARY_DIR}/webrtc-extras.cmake)
execute_process(
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} generate-cmake-extras ${CMAKE_EXTRAS_FILE} ${CATKIN_DEVEL_PREFIX}/${CATKIN_GLOBAL_LIB_DESTINATION} # this must come before catkin_packate, which sets up CATKIN_PACKAGE_* macros so use global instead of package
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
  RESULT_VARIABLE CMAKE_EXTRAS_RESULT
)
if(NOT ${CMAKE_EXTRAS_RESULT} EQUAL 0)
  message(FATAL_ERROR "WebRTC cmake extras generation failed")
endif()


catkin_package(
  CFG_EXTRAS ${CMAKE_EXTRAS_FILE}
)

#############
##  Build  ##
#############

add_custom_target(webrtc ALL
  COMMAND ./build_webrtc.sh ${webrtc_build_root}
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} copy-libs ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} copy-tools ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
)

#############
## Install ##
#############

# Generate list of built libraries
execute_process(
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} list-built-libs ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION}
  OUTPUT_VARIABLE BUILT_LIBRARIES
  OUTPUT_STRIP_TRAILING_WHITESPACE
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
  RESULT_VARIABLE LIST_BUILT_LIBS_RESULT
)
if(NOT ${LIST_BUILT_LIBS_RESULT} EQUAL 0)
  message(FATAL_ERROR "Listing built WebRTC libs failed")
endif()

# Install libraries
install(FILES ${BUILT_LIBRARIES}
  DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

# Install tools
install(PROGRAMS
  ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}/stunserver
  ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}/turnserver
  ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_BIN_DESTINATION}/relayserver
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)


# Install license files
install(FILES
  ../LICENSE
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(FILES
  src/webrtc/LICENSE
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  RENAME LICENSE_webrtc
)
install(FILES
  src/webrtc/LICENSE_THIRD_PARTY
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  RENAME LICENSE_THIRD_PARTY_webrtc
)
install(FILES
  src/talk/LICENSE_THIRD_PARTY
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  RENAME LICENSE_THIRD_PARTY_talk
)

# Install includes
install(CODE "
execute_process(
  COMMAND ./ninja_to_cmake.py ${webrtc_build_root} install-includes ${CMAKE_INSTALL_PREFIX}/${CATKIN_GLOBAL_INCLUDE_DESTINATION}
  WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/
  RESULT_VARIABLE INSTALL_INCLUDES_RESULT
)
if(NOT ${INSTALL_INCLUDES_RESULT} EQUAL 0)
  message(FATAL_ERROR \"Installing WebRTC includes failed\")
endif(NOT ${INSTALL_INCLUDES_RESULT} EQUAL 0)
")
