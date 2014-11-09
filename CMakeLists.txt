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


# oglplus provides a nice interface for OpenGL encapsulation in C++
#
# However, it's CMake setup isn't well suited for inclusion in 
# another project the way I'm doing.  Most importantly, it does 
# compiler detection in the CMake setup and then configures a 
# header file with a bunch of preprocessor macros based on what it 
# found.  
#
# In order to support using it as a header only library, I need to
# set a bunch of values here, then change the 'PROJECT_SOURCE_DIR' 
# to the oglplus folder (because it's compiler detection stuff 
# uses a bunch of files that are considered relative to said dir)
# and then restore the normal project directory.  

# Set up the configuration constants
set(OGLPLUS_CONFIG_SET_LOW_PROFILE 1)
set(OGLPLUS_LOW_PROFILE 0)
set(OGLPLUS_USE_GLCOREARB_H 0) 
set(OGLPLUS_USE_GL3_H 0)
set(OGLPLUS_USE_GLEW 1)
set(OGLPLUS_USE_GL3W 0)
set(OGLPLUS_USE_BOOST_CONFIG 0)

# Save and change the PROJECT_SOURCE_DIR, so the oglplus 
# compiler and C++ feature detection code will work
set(SAVE_PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR})
set(PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/libraries/oglplus)
# Run compiler and C++ feature detection code
include(libraries/oglplus/config/Compiler.cmake)
include(libraries/oglplus/config/CPPFeature.cmake)
# Restore our original project source directory
set(PROJECT_SOURCE_DIR ${SAVE_PROJECT_SOURCE_DIR})
# Generate the oglplus 'site' header file which switches
# code paths based on detected C++ features.
configure_file(
	${PROJECT_SOURCE_DIR}/libraries/oglplus/config/oglplus/config/site.hpp.in
	${PROJECT_BINARY_DIR}/libraries/oglplus/include/oglplus/config/site.hpp
)

# Add the required oglplus include paths
include_directories(${CMAKE_SOURCE_DIR}/libraries/oglplus/include)
include_directories(${CMAKE_SOURCE_DIR}/libraries/oglplus/implement)
# The site config header is in the binary directory, so we need to 
# add that path too
include_directories(${CMAKE_BINARY_DIR}/libraries/oglplus/include)

###############################################################################
#
# GLM - Vector / matrix header only math library based on the syntax of GLSL
#
# It's cleverly puts all it's compiler and C++ feature  detection code into a
# header that switches based on what compiler is being used (based on predefined 
# macros).  Why can't everyone do that?
include_directories(${CMAKE_SOURCE_DIR}/libraries/glm)

###############################################################################
#
# Oculus C API 
#
# This provides all the access we need to the Oculus SDK.  For some strange 
# reason the header files are in LibOVR/Src instead of LibOVR/Include.  
#
include_directories(${CMAKE_SOURCE_DIR}/libraries/OculusSDK/LibOVR/Src)

###############################################################################
#
# The examples themselves
#
add_subdirectory(examples/cpp)
