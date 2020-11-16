project(tvm_vendor)

find_package(ros_environment REQUIRED)

set(ROS_VERSION $ENV{ROS_VERSION})

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

set(tvm_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/tvm_install)

include(ExternalProject)
ExternalProject_Add(tvm
  GIT_REPOSITORY https://github.com/apache/incubator-tvm
  GIT_TAG v0.7.0
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE TRUE
  INSTALL_DIR ${tvm_DESTDIR}
  PATCH_COMMAND git apply ${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.patch
  CONFIGURE_COMMAND ${CMAKE_COMMAND} -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  INSTALL_COMMAND ""
)

file(GLOB SHARED_LIBS
  "${tvm_DESTDIR}/lib/*.so"
)

set(SHORT_LIBS "")

foreach(SHARED_LIB ${SHARED_LIBS})
  string(REPLACE ${tvm_DESTDIR}/lib/ "" SHORT_LIB ${SHARED_LIB})
  list(APPEND SHORT_LIBS ${SHORT_LIB})
endforeach()

if(${ROS_VERSION} EQUAL 1)
  cmake_minimum_required(VERSION 2.8.3)

  # Default to C++11
  if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 14)
  endif()

  # find dependencies
  find_package(catkin REQUIRED)

  catkin_package(
    INCLUDE_DIRS include
    LIBRARIES ${SHORT_LIBS}
  )

  install(DIRECTORY ${tvm_DESTDIR}/include/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  )

  install(DIRECTORY ${tvm_DESTDIR}/lib/
    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  )
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

  ament_export_include_directories(include)
  ament_export_libraries(${SHORT_LIBS})

  install(DIRECTORY ${tvm_DESTDIR}/include/
    DESTINATION include
  )

  install(DIRECTORY ${tvm_DESTDIR}/lib/
    DESTINATION lib
  )

  ament_package()
endif()
