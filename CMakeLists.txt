cmake_minimum_required(VERSION 3.5)
project(tvm_vendor)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)

set(tvm_DESTDIR ${CMAKE_CURRENT_BINARY_DIR}/tvm_install)

include(ExternalProject)
ExternalProject_Add(tvm
  GIT_REPOSITORY https://github.com/apache/incubator-tvm
  GIT_TAG v0.7.0
  GIT_SHALLOW TRUE
  BUILD_IN_SOURCE TRUE
  INSTALL_DIR ${tvm_DESTDIR}
  PATCH_COMMAND git apply ${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.patch
  CONFIGURE_COMMAND cp build/config.cmake .; ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR> -DCMAKE_POSITION_INDEPENDENT_CODE=ON
)

file(GLOB SHARED_LIBS
  "${tvm_DESTDIR}/lib/*.so"
)

set(SHORT_LIBS "")

foreach(SHARED_LIB ${SHARED_LIBS})
  string(REPLACE ${tvm_DESTDIR}/lib/ "" SHORT_LIB ${SHARED_LIB})
  list(APPEND SHORT_LIBS ${SHORT_LIB})
endforeach()

ament_export_include_directories(include)
ament_export_libraries(${SHORT_LIBS})

install(DIRECTORY ${tvm_DESTDIR}/include/
  DESTINATION include
)

install(DIRECTORY ${tvm_DESTDIR}/lib/
  DESTINATION lib
)

ament_package()
