cmake_minimum_required(VERSION 3.0.2)
project(GO2_tasks)

## 编译选项
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

## 设置默认编译类型
set(default_build_type "Release")
if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
            STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

## 查找catkin依赖
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
)

## 定义catkin包
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES unitree_sdk2
)

## 添加包含路径
include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

## 检查系统架构
message(STATUS "Current system architecture: ${CMAKE_SYSTEM_PROCESSOR}")

## 导入第三方库
add_subdirectory(thirdparty)

## 导入 Unitree SDK2 库
set(UNITREE_SDK_PATH ${CMAKE_CURRENT_LIST_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR})
find_library(UNITREE_SDK_LIB unitree_sdk2 PATHS ${UNITREE_SDK_PATH} NO_DEFAULT_PATH)

if (NOT UNITREE_SDK_LIB)
    message(FATAL_ERROR "Unitree SDK library for the architecture is not found")
else ()
    message(STATUS "Unitree SDK library found at: ${UNITREE_SDK_LIB}")
endif ()

add_library(unitree_sdk2 STATIC IMPORTED GLOBAL)
set_target_properties(unitree_sdk2 PROPERTIES
        IMPORTED_LOCATION ${UNITREE_SDK_LIB})
target_link_libraries(unitree_sdk2 INTERFACE ddsc ddscxx ${catkin_LIBRARIES})

## 定义可执行文件并作为ROS节点
add_executable(go2_sport_client /home/mingtao/GO2_ws/src/GO2_tasks/example/go2/go2_sport_client.cpp)
target_link_libraries(go2_sport_client ${catkin_LIBRARIES} unitree_sdk2)

# 根据其他可执行文件定义类似规则
# 示例:
# add_executable(node_name src/file_name.cpp)
# target_link_libraries(node_name ${catkin_LIBRARIES} unitree_sdk2)

## 安装目标
install(TARGETS go2_sport_client
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

## 安装头文件
install(DIRECTORY include/
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

## 安装库
install(FILES ${UNITREE_SDK_LIB}
    DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

