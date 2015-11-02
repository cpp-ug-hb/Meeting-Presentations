add_library( print STATIC
  print.cpp
  print.hpp
)
target_include_directories( print PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} )
