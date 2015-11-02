cmake_minimum_required(VERSION 3.0)
project( main )

find_package( Print 1.0 REQUIRED )

add_executable( hello main.cpp )
target_link_libraries( hello print )
install( TARGETS hello DESTINATION bin )
