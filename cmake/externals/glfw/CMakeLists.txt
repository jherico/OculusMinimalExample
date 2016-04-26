include(ExternalProject)
include(SelectLibraryConfigurations)

set(EXTERNAL_NAME glfw)

string(TOUPPER ${EXTERNAL_NAME} EXTERNAL_NAME_UPPER)

if (WIN32)

  if ("${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
      ExternalProject_Add(
        ${EXTERNAL_NAME}
        URL https://s3.amazonaws.com/Oculus/glfw-3.1.2.bin.WIN64.zip
        URL_MD5 77189f3105f7fb2d34b1cf1d8e340ab2
        CONFIGURE_COMMAND ""
        BUILD_COMMAND ""
        INSTALL_COMMAND ""
        LOG_DOWNLOAD 1
      )
  else()
    message(FATAL_ERROR "32 bit builds not supported")
  endif()


  ExternalProject_Get_Property(${EXTERNAL_NAME} SOURCE_DIR)
  set(${EXTERNAL_NAME_UPPER}_INCLUDE_DIRS ${SOURCE_DIR}/include CACHE TYPE INTERNAL)
  set(${EXTERNAL_NAME_UPPER}_LIBRARIES ${SOURCE_DIR}/lib-vc2013/glfw3.lib CACHE TYPE INTERNAL)

else()
  message(FATAL_ERROR "Non-windows builds not supported")
endif()

# Hide this external target (for ide users)
set_target_properties(${EXTERNAL_NAME} PROPERTIES FOLDER "hidden/externals")


