cmake_minimum_required(VERSION 3.9)

project(triangle LANGUAGES CXX C VERSION 1.0.0)

# Set C++ Standard
###########################################################################################
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Create library variables
###############################################################################################
set(TRIANGLE_LIBRARY_SOURCES ${PROJECT_NAME}_sources)
set(TRIANGLE_LIBRARY_HEADERS ${PROJECT_NAME}_headers)
set(TRIANGLE_LIBRARY_PUBLIC_HEADERS ${PROJECT_NAME}_public_headers)
set(TRIANGLE_LIBRARY_LINKED_LIBRARIES ${PROJECT_NAME}_linked_libraries)
set(TRIANGLE_LIBRARY_INCLUDE ${PROJECT_NAME}_include)
set(TRIANGLE_LIBRARY_ADDITIONAL_INCLUDE ${PROJECT_NAME}_additional_include)
set(TRIANGLE_INSTALL_DIR ${PROJECT_NAME})
set(TRIANGLE_INSTALL_BINARY_DIR bin/)
set(TRIANGLE_INSTALL_INCLUDE_DIR include/)
set(TRIANGLE_INSTALL_LIB_DIR lib/)
set(TRIANGLE_INSTALL_ARCHIVE_DIR lib/)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()
Message(STATUS "Triangle Build configuration: " ${CMAKE_BUILD_TYPE})

# Add dependencies
###########################################################################################
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Insert Sources
###############################################################################################
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/src)
list(APPEND ${TRIANGLE_LIBRARY_SOURCES} ${triangle_src_sources})
list(APPEND ${TRIANGLE_LIBRARY_HEADERS} ${triangle_src_headers})
list(APPEND ${TRIANGLE_LIBRARY_INCLUDE} ${triangle_src_includes})

# Create public headers
###############################################################################################
set(TRIANGLE_LIBRARY_ADDITIONAL_INCLUDE_PATH ${CMAKE_CURRENT_BINARY_DIR}/${TRIANGLE_LIBRARY_ADDITIONAL_INCLUDE})

foreach(header ${${TRIANGLE_LIBRARY_HEADERS}})
        get_filename_component(headerDirectory ${header} DIRECTORY)
        get_filename_component(headerFileName ${header} NAME_WE)
        get_filename_component(headerExtension ${header} EXT)

  if(${headerExtension} MATCHES ".hpp.in")
    set(headerExtension ".hpp")
    set(headerDirectory ${TRIANGLE_LIBRARY_ADDITIONAL_INCLUDE_PATH})

    configure_file(${header} ${headerDirectory}/${headerFileName}${headerExtension} @ONLY)
  elseif(NOT ${headerExtension} MATCHES ".hpp" AND NOT ${headerExtension} MATCHES ".h")
    message(SEND_ERROR "Header extension not recognized for file ${headerFileName}${headerExtension}")
  endif()

  list(APPEND ${TRIANGLE_LIBRARY_PUBLIC_HEADERS} ${headerDirectory}/${headerFileName}${headerExtension})
endforeach()

list(APPEND ${TRIANGLE_LIBRARY_INCLUDE} ${TRIANGLE_LIBRARY_ADDITIONAL_INCLUDE_PATH})

# Create library
###############################################################################################
add_library(${PROJECT_NAME} STATIC ${${TRIANGLE_LIBRARY_SOURCES}} ${${TRIANGLE_LIBRARY_HEADERS}})
add_library(triangle::triangle ALIAS ${PROJECT_NAME})

set_property(SOURCE ${${TRIANGLE_LIBRARY_SOURCES}} PROPERTY LANGUAGE C )
set_target_properties(${PROJECT_NAME} PROPERTIES "COMPILE_DEFINITIONS" TRILIBRARY)

target_link_libraries(${PROJECT_NAME} ${${TRIANGLE_LIBRARY_LINKED_LIBRARIES}})

target_include_directories(${PROJECT_NAME} PRIVATE ${${TRIANGLE_LIBRARY_INCLUDE}})
target_include_directories(${PROJECT_NAME} PUBLIC $<INSTALL_INTERFACE:${TRIANGLE_INSTALL_INCLUDE_DIR}>)

target_compile_options(${PROJECT_NAME} PUBLIC -fPIC)

# Create install target
###############################################################################################
set_target_properties(${PROJECT_NAME} PROPERTIES 
PUBLIC_HEADER "${${TRIANGLE_LIBRARY_PUBLIC_HEADERS}}" 
EXPORT_NAME triangle)

install(
	TARGETS ${PROJECT_NAME} 
	EXPORT triangleTargets
    RUNTIME DESTINATION ${TRIANGLE_INSTALL_BINARY_DIR} # runtime executable
    LIBRARY DESTINATION ${TRIANGLE_INSTALL_LIB_DIR} # dynamic libraries
    ARCHIVE DESTINATION ${TRIANGLE_INSTALL_ARCHIVE_DIR} # static libraries
    PUBLIC_HEADER DESTINATION ${TRIANGLE_INSTALL_INCLUDE_DIR} # headers
)


include(CMakePackageConfigHelpers)

configure_package_config_file(
	${CMAKE_CURRENT_SOURCE_DIR}/cmake/triangleConfig.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/triangleConfig.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/${TRIANGLE_INSTALL_LIB_DIR}/cmake
	PATH_VARS TRIANGLE_INSTALL_INCLUDE_DIR
  )

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/triangleConfigVersion.cmake
	VERSION ${triangle_VERSION}
	COMPATIBILITY AnyNewerVersion
)

export(
	TARGETS ${PROJECT_NAME} 
	NAMESPACE triangle::
	FILE ${CMAKE_CURRENT_BINARY_DIR}/triangleTargets.cmake
)

install(
	EXPORT triangleTargets 
	NAMESPACE triangle:: 
	DESTINATION ${CMAKE_INSTALL_PREFIX}/${TRIANGLE_INSTALL_LIB_DIR}/cmake
)

install(
	FILES
		${CMAKE_CURRENT_BINARY_DIR}/triangleConfig.cmake
		${CMAKE_CURRENT_BINARY_DIR}/triangleConfigVersion.cmake
	DESTINATION
		${CMAKE_INSTALL_PREFIX}/${TRIANGLE_INSTALL_LIB_DIR}/cmake
)
