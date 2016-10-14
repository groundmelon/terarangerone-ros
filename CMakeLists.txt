cmake_minimum_required(VERSION 2.8.3)
project(terarangerone)

set(CMAKE_VERBOSE_MAKEFILE "true")
include(CheckCXXCompilerFlag)

check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_STD_CXX11)
if (NOT COMPILER_SUPPORTS_STD_CXX11)
  check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_STD_CXX0X)
endif ()

if (COMPILER_SUPPORTS_STD_CXX11)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif (COMPILER_SUPPORTS_STD_CXX0X)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else ()
  message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif ()

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -g -march=native -Wno-unused-result -funroll-loops")


# search for everything we need to build the package
find_package(catkin REQUIRED COMPONENTS
  roscpp
  sensor_msgs
  std_msgs
  dynamic_reconfigure
)
# since we need boost search it as well
# find_package makes the ${..._INCLUDE_DIRS} ${..._LIBRARIES} variables we use later
find_package(Boost REQUIRED COMPONENTS system thread)

generate_dynamic_reconfigure_options(
  cfg/TerarangerOne.cfg
)

# export the dependencis of this package for who ever depends on us
catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS roscpp sensor_msgs std_msgs dynamic_reconfigure
  DEPENDS boost
)

# tell catkin where to find the headers for this project
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)



# make the executables
add_executable( terarangerone_node src/terarangerone.cpp src/serial_port.cpp  )
add_dependencies( terarangerone_node  ${catkin_EXPORTED_TARGETS} )
add_dependencies(terarangerone_node ${PROJECT_NAME}_gencfg)
target_link_libraries( terarangerone_node ${catkin_LIBRARIES} ${Boost_LIBRARIES} )

