cmake_minimum_required(VERSION 3.5)
project(rosbag2_compression)

# Declare carma package and check ROS version
find_package(carma_cmake_common REQUIRED)
carma_check_ros_version(2)
carma_package()

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

# Windows supplies macros for min and max by default. We should only use min and max from stl
if(WIN32)
  add_definitions(-DNOMINMAX)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rcpputils REQUIRED)
find_package(rcutils REQUIRED)
find_package(rosbag2_cpp)
find_package(rosbag2_storage REQUIRED)

add_library(${PROJECT_NAME}
  SHARED
  src/rosbag2_compression/compression_factory.cpp
  src/rosbag2_compression/compression_options.cpp
  src/rosbag2_compression/sequential_compression_reader.cpp
  src/rosbag2_compression/sequential_compression_writer.cpp)
target_include_directories(${PROJECT_NAME}
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
ament_target_dependencies(${PROJECT_NAME}
  pluginlib
  rcpputils
  rcutils
  rosbag2_cpp
  rosbag2_storage)
target_compile_definitions(${PROJECT_NAME} PRIVATE ROSBAG2_COMPRESSION_BUILDING_DLL)

# prevent pluginlib from using boost
target_compile_definitions(${PROJECT_NAME} PUBLIC "PLUGINLIB__DISABLE_BOOST_FUNCTIONS")

install(
  DIRECTORY include/
  DESTINATION include)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_export_targets(export_${PROJECT_NAME})
ament_export_dependencies(pluginlib rcpputils rcutils rosbag2_cpp rosbag2_storage)

if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(ament_lint_auto REQUIRED)
  find_package(rclcpp REQUIRED)
  find_package(rosbag2_test_common REQUIRED)
  ament_lint_auto_find_test_dependencies()

  add_library(fake_plugin SHARED
    test/rosbag2_compression/fake_compressor.cpp
    test/rosbag2_compression/fake_decompressor.cpp)
  target_link_libraries(fake_plugin ${PROJECT_NAME})
  # Needs to be removed/revised after https://github.com/ros2/ros2/issues/1150 gets fixed
  ament_target_dependencies(fake_plugin rosbag2_cpp rosbag2_storage)
  install(
    TARGETS fake_plugin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin)
  pluginlib_export_plugin_description_file(
    rosbag2_compression test/rosbag2_compression/fake_plugin.xml)

  ament_add_gmock(test_compression_factory
    test/rosbag2_compression/test_compression_factory.cpp)
  target_link_libraries(test_compression_factory ${PROJECT_NAME})
  # Needs to be removed/revised after https://github.com/ros2/ros2/issues/1150 gets fixed
  ament_target_dependencies(test_compression_factory rosbag2_cpp rosbag2_storage)

  ament_add_gmock(test_compression_options
    test/rosbag2_compression/test_compression_options.cpp)
  target_include_directories(test_compression_options PUBLIC include)
  target_link_libraries(test_compression_options ${PROJECT_NAME})
  # Needs to be removed/revised after https://github.com/ros2/ros2/issues/1150 gets fixed
  ament_target_dependencies(test_compression_options rosbag2_cpp rosbag2_storage)

  ament_add_gmock(test_sequential_compression_reader
    test/rosbag2_compression/test_sequential_compression_reader.cpp)
  target_include_directories(test_sequential_compression_reader PUBLIC include)
  target_link_libraries(test_sequential_compression_reader ${PROJECT_NAME})
  # Needs to be removed/revised after https://github.com/ros2/ros2/issues/1150 gets fixed
  ament_target_dependencies(test_sequential_compression_reader rosbag2_cpp rosbag2_storage)

  ament_add_gmock(test_sequential_compression_writer
    test/rosbag2_compression/test_sequential_compression_writer.cpp)
  target_include_directories(test_sequential_compression_writer PUBLIC include)
  target_link_libraries(test_sequential_compression_writer ${PROJECT_NAME})
  # Needs to be removed/revised after https://github.com/ros2/ros2/issues/1150 gets fixed
  ament_target_dependencies(test_sequential_compression_writer rosbag2_cpp rosbag2_storage)
endif()

ament_package()
