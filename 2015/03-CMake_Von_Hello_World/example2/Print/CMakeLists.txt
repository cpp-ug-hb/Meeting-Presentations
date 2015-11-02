cmake_minimum_required(VERSION 3.0)
project(Print)


## Build the library
add_library(
  print STATIC
  print.cpp
  print.hpp
)

## Install it and the header
install(
  TARGETS print
  EXPORT Print-export
  DESTINATION lib
  INCLUDES DESTINATION include
)

install(
  FILES print.hpp
  DESTINATION include
)

if(UNIX)
  set(CONFIG_DIR lib/Print/cmake)
else()
  set(CONFIG_DIR CMake)
endif()

## Generate and install CMake config files
include(CMakePackageConfigHelpers)

configure_package_config_file(
  PrintConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/PrintConfig.cmake
  INSTALL_DESTINATION ${CONFIG_DIR}
)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/PrintConfigVersion.cmake"
  VERSION 1.0
  COMPATIBILITY SameMajorVersion
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/PrintConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/PrintConfigVersion.cmake
  DESTINATION ${CONFIG_DIR}
)

install(
  EXPORT Print-export
  DESTINATION lib/Print/cmake
)
