cmake_minimum_required(VERSION 3.1)
project( example1 )

add_subdirectory( libprint )

add_executable( hello main.cpp )
target_link_libraries( hello print )
install( TARGETS hello DESTINATION bin )
