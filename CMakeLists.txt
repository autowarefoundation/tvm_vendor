project(tvm_vendor)

find_package(ros_environment REQUIRED)

set(ROS_VERSION $ENV{ROS_VERSION})

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

if(${ROS_VERSION} EQUAL 1)
  cmake_minimum_required(VERSION 2.8.3)

  # Default to C++11
  if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
  endif()

  # find dependencies
  find_package(catkin REQUIRED)
elseif(${ROS_VERSION} EQUAL 2)
  cmake_minimum_required(VERSION 3.5)

  # Default to C++14
  if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
  endif()

  if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    add_compile_options(-Wall -Wextra -Wpedantic)
  endif()

  # find dependencies
  find_package(ament_cmake REQUIRED)
endif()

include(ExternalProject)

ExternalProject_Add(tvm-build
  GIT_REPOSITORY https://github.com/apache/incubator-tvm
  GIT_TAG v0.7.0
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE TRUE
  PATCH_COMMAND git apply ${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.patch
  COMMAND cp <SOURCE_DIR>/cmake/config.cmake <SOURCE_DIR>
  CONFIGURE_COMMAND ${CMAKE_COMMAND} -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  INSTALL_COMMAND ""
)

ExternalProject_Get_property(tvm-build SOURCE_DIR)

add_library(tvm SHARED IMPORTED
  IMPORTED_LOCATION ${SOURCE_DIR}/libtvm.so
)

add_library(tvm_runtime SHARED IMPORTED
  IMPORTED_LOCATION ${SOURCE_DIR}/libtvm_runtime.so
)

if(${ROS_VERSION} EQUAL 1)
  catkin_package(
    INCLUDE_DIRS include
    LIBRARIES tvm tvm_runtime
  )

  add_custom_command(TARGET tvm-build POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_SOURCE_DIR}/include/tvm
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${SOURCE_DIR}/include/tvm/runtime/ ${CMAKE_CURRENT_SOURCE_DIR}/include/tvm/runtime
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CATKIN_DEVEL_PREFIX}/lib
    COMMAND ${CMAKE_COMMAND} -E copy ${SOURCE_DIR}/libtvm.so ${CATKIN_DEVEL_PREFIX}/lib
    COMMAND ${CMAKE_COMMAND} -E copy ${SOURCE_DIR}/libtvm_runtime.so ${CATKIN_DEVEL_PREFIX}/lib
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_SOURCE_DIR}/include/dlpack
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${SOURCE_DIR}/3rdparty/dlpack/include/dlpack/dlpack.h ${CMAKE_CURRENT_SOURCE_DIR}/include/dlpack/dlpack.h
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${SOURCE_DIR}/3rdparty/dlpack/contrib/dlpack/dlpackcpp.h ${CMAKE_CURRENT_SOURCE_DIR}/include/dlpack/dlpackcpp.h
  )

  install(DIRECTORY ${SOURCE_DIR}/include/tvm/runtime
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/tvm
  )

  install(
    FILES
      ${SOURCE_DIR}/libtvm.so
      ${SOURCE_DIR}/libtvm_runtime.so
    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  )

  install(
    FILES
      ${SOURCE_DIR}/3rdparty/dlpack/include/dlpack/dlpack.h
      ${SOURCE_DIR}/3rdparty/dlpack/contrib/dlpack/dlpackcpp.h
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}/dlpack
  )
elseif(${ROS_VERSION} EQUAL 2)
  ament_export_include_directories(include)
  ament_export_libraries(tvm tvm_runtime)

  install(DIRECTORY ${SOURCE_DIR}/include/tvm/runtime/
    DESTINATION include/${PROJECT_NAME}/tvm/runtime
  )

  install(
    FILES
      ${SOURCE_DIR}/libtvm.so
      ${SOURCE_DIR}/libtvm_runtime.so
    DESTINATION lib
  )

  install(
    FILES
      ${SOURCE_DIR}/3rdparty/dlpack/include/dlpack/dlpack.h
      ${SOURCE_DIR}/3rdparty/dlpack/contrib/dlpack/dlpackcpp.h
    DESTINATION include/${PROJECT_NAME}/dlpack
  )

  ament_package()
endif()
