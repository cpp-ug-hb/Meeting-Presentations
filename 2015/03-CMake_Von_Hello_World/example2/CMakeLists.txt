cmake_minimum_required(VERSION 3.0)
project(example2)


#####
# Scripted building of Print and then main
#####

include(ExternalProject)

ExternalProject_Add(
  Print
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/Print
  INSTALL_DIR ${CMAKE_BINARY_DIR}/install/Print
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
)

ExternalProject_Add(
  main
  SOURCE_DIR ${CMAKE_SOURCE_DIR}/main
  INSTALL_DIR ${CMAKE_BINARY_DIR}/install/main
  CMAKE_ARGS
    -DCMAKE_PREFIX_PATH=${CMAKE_BINARY_DIR}/install/Print
    -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
    -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
  DEPENDS Print
)
