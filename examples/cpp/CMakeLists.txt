###############################################################################
#
# Shared codebase for all the examples and demos
#

function(make_example FILE PROJECT_FOLDER)
    set(SOURCE_FILES ${FILE})
    file(RELATIVE_PATH FILE ${CMAKE_CURRENT_SOURCE_DIR} ${FILE})
    get_filename_component(NAME ${FILE} NAME_WE)
    string(REGEX REPLACE "^([^_]+(_[0-9]+)+).*\\.cpp$" "\\1" ROOT ${FILE} )

    set(EXECUTABLE "${NAME}")

    if(WIN32)
        add_executable(${EXECUTABLE} WIN32 ${SOURCE_FILES})
        list(APPEND EXAMPLE_LIBS d3d11)
        list(APPEND EXAMPLE_LIBS dxgi)
    elseif(APPLE)
        list(APPEND SOURCE_FILES ${ALL_RESOURCES})
        add_executable(${EXECUTABLE} MACOSX_BUNDLE ${SOURCE_FILES} )
    else()
        add_executable(${EXECUTABLE} ${SOURCE_FILES} )
    endif()

    target_link_libraries(${EXECUTABLE} ${EXAMPLE_LIBS})
	if(BUILD_TYPE_LOWER STREQUAL "debug")
        set_property(TARGET ${EXECUTABLE} PROPERTY DEBUG_OUTPUT_NAME ${EXECUTABLE}_d)
	endif()

    set_target_properties(${EXECUTABLE} PROPERTIES FOLDER ${PROJECT_FOLDER})
endfunction()

function(make_examples GLOB PROJECT_FOLDER)
    file(GLOB EXAMPLE_APPS ${GLOB})
    list(SORT EXAMPLE_APPS)
    foreach (FILE ${EXAMPLE_APPS})
        make_example(${FILE} ${PROJECT_FOLDER})
    endforeach()
endfunction() 

make_examples(*.cpp "Examples")


