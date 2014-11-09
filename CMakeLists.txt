# Required for statically linking the MSVC runtime
set(CMAKE_USER_MAKE_RULES_OVERRIDE
   ${CMAKE_CURRENT_SOURCE_DIR}/cmake/c_flag_overrides.cmake)
set(CMAKE_USER_MAKE_RULES_OVERRIDE_CXX
   ${CMAKE_CURRENT_SOURCE_DIR}/cmake/cxx_flag_overrides.cmake)

project(OculusMinimalExample)
cmake_minimum_required(VERSION 2.8)
include(GenerateExportHeader)
include(cmake/defaults.cmake)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

string( TOLOWER "${CMAKE_BUILD_TYPE}" BUILD_TYPE_LOWER )
if(BUILD_TYPE_LOWER STREQUAL "debug")
   set(RIFT_DEBUG 1)
endif()

###############################################################################
#
# Oculus VR SDK dependency 
#
set(OVR_DIR ${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVR)
# Check for the existence of the LibOVR project.  If it's missing, the 
# submodule probably hasn't been checked out.
if (NOT(EXISTS ${OVR_DIR} AND IS_DIRECTORY ${OVR_DIR}))
    message(FATAL_ERROR "Submodules seem to be missing.  Please make sure you check out with 'git clone --recursive' OR run 'git submodule init' and 'git submodule update' after checkout") 
endif()

# The primary SDK artifact, a static library for Oculus access 
set(BUILD_SHARED_LIBS ON)
add_subdirectory(libraries/OculusSDK/LibOVR)
set(BUILD_SHARED_LIBS OFF)
set_target_properties(OculusVR PROPERTIES FOLDER "3rdparty")
list(APPEND EXAMPLE_LIBS OculusVR)


###############################################################################
#
# Non-Oculus third party dependencies 
#

# GLEW - Cross platform access to OpenGL 3+ functionality
add_definitions(-DGLEW_STATIC)
add_subdirectory(libraries/glew)
set_target_properties(glew PROPERTIES FOLDER "3rdparty")
list(APPEND EXAMPLE_LIBS glew)

# GLFW - Cross platform OpenGL window creation and input handling
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "Build the GLFW example programs")
set(GLFW_BUILD_TESTS OFF CACHE BOOL "Build the GLFW test programs")
set(GLFW_BUILD_DOCS OFF CACHE BOOL "Build the GLFW documentation")
set(GLFW_INSTALL OFF CACHE BOOL "Generate installation target")
add_subdirectory(libraries/glfw)
set_target_properties(glfw PROPERTIES FOLDER "3rdparty")
list(APPEND EXAMPLE_LIBS glfw ${GLFW_LIBRARIES})

find_package(Threads)
list(APPEND EXAMPLE_LIBS ${CMAKE_THREAD_LIBS_INIT} )

include_directories(${CMAKE_SOURCE_DIR}/libraries/glew/include)
include_directories(${CMAKE_SOURCE_DIR}/libraries/glfw/include)
include_directories(${CMAKE_SOURCE_DIR}/libraries/glm)

# OpenGL encapsulation
set(OGLPLUS_CONFIG_SET_LOW_PROFILE 1)
set(OGLPLUS_LOW_PROFILE 0)
set(OGLPLUS_USE_GLCOREARB_H 0) 
set(OGLPLUS_USE_GL3_H 0)
set(OGLPLUS_USE_GLEW 1)
set(OGLPLUS_USE_GL3W 0)
set(OGLPLUS_USE_BOOST_CONFIG 0)

set(SAVE_PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR})
set(PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/libraries/oglplus)
#message(FATAL_ERROR ${PROJECT_SOURCE_DIR})
# compiler options
include(libraries/oglplus/config/Compiler.cmake)
# c++ compiler options / supported feature detection
include(libraries/oglplus/config/CPPFeature.cmake)
set(PROJECT_SOURCE_DIR ${SAVE_PROJECT_SOURCE_DIR})

configure_file(
	${PROJECT_SOURCE_DIR}/libraries/oglplus/config/oglplus/config/site.hpp.in
	${PROJECT_BINARY_DIR}/libraries/oglplus/include/oglplus/config/site.hpp
)

#add_subdirectory(libraries/oglplus)


include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVR/Src)
include_directories(${CMAKE_SOURCE_DIR}/libraries/oglplus/include)
include_directories(${CMAKE_SOURCE_DIR}/libraries/oglplus/implement)
include_directories(${CMAKE_BINARY_DIR}/libraries/oglplus/include)

###############################################################################
#
# GLM - Vector / matrix header only math library based on the syntax of GLSL
#
include_directories(${CMAKE_SOURCE_DIR}/libraries/glm)


###############################################################################
#
# The examples themselves
#
foreach(LIB ${EXAMPLE_LIBS}) 
	message(${LIB})
endforeach()
add_subdirectory(examples/cpp)
