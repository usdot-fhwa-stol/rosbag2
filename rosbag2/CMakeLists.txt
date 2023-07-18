cmake_minimum_required(VERSION 3.5)
project(rosbag2)

# Declare carma package and check ROS version
find_package(carma_cmake_common REQUIRED)
carma_check_ros_version(2)
carma_package()

find_package(ament_cmake REQUIRED)
ament_package()
