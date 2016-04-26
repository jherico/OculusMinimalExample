cmake_minimum_required(VERSION 3.2)
include(cmake/defaults.cmake)

set(MAIN_APP_NAME OculusMinimalExample)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(GLOBAL PROPERTY PREDEFINED_TARGETS_FOLDER "CMakeTargets")

set(TARGET_NAME ${MAIN_APP_NAME})
project(${TARGET_NAME})

set(APP_SRCS OculusMinimalExample.cpp)

# create the executable, make it a bundle on OS X
add_executable(${TARGET_NAME} WIN32 ${APP_SRCS})

target_glfw()
target_glew()
target_opengl()
target_glm()
target_oglplus()
add_dependency_external_projects(LibOVR)
find_package(LibOVR REQUIRED)
target_include_directories(${TARGET_NAME} PRIVATE ${LIBOVR_INCLUDE_DIRS})
target_link_libraries(${TARGET_NAME} ${LIBOVR_LIBRARIES})


