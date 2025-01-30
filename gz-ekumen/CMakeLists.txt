cmake_minimum_required(VERSION 3.7)
project(gz-ekumen)

find_package(ament_cmake REQUIRED)

install(
  DIRECTORY
    models
    worlds
  DESTINATION
    share/${PROJECT_NAME}/
)

ament_environment_hooks("${CMAKE_CURRENT_SOURCE_DIR}/env-hooks/${PROJECT_NAME}.dsv")
ament_environment_hooks("${CMAKE_CURRENT_SOURCE_DIR}/env-hooks/${PROJECT_NAME}.sh")

ament_package()
