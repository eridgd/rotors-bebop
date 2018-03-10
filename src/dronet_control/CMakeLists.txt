cmake_minimum_required(VERSION 2.8.3)
project(dronet_control)

find_package(catkin_simple REQUIRED)
catkin_simple()

set(CMAKE_CXX_FLAGS "-std=c++11")

cs_add_executable(dronet_control src/deep_navigation.cpp)

cs_export()


