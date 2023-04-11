cmake_minimum_required(VERSION 3.9)

project(metis LANGUAGES C VERSION 5.1.1)

# Set C/C++ Standard
###########################################################################################
set(CMAKE_C_STANDARD 99)

# Create library variables
###############################################################################################
set(METIS_INSTALL_DIR ${PROJECT_NAME})
set(METIS_INSTALL_BINARY_DIR bin/)
set(METIS_INSTALL_INCLUDE_DIR include/)
set(METIS_INSTALL_LIB_DIR lib/)
set(METIS_INSTALL_ARCHIVE_DIR lib/)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel" FORCE)
endif()
Message(STATUS "METIS Build configuration: " ${CMAKE_BUILD_TYPE})

# Configure libmetis library.
option(BUILD_TESTING "Build METIS self-tests" OFF)

if(NOT intsize)
  set(intsize 32)
endif()
if(NOT realsize)
  set(realsize 64)
endif()

if(BUILD_SHARED_LIBS)
  set(METIS_LIBRARY_TYPE SHARED)
else()
  set(METIS_LIBRARY_TYPE STATIC)
endif()

set(GKLIB_PATH ${CMAKE_CURRENT_SOURCE_DIR}/src/GKlib CACHE PATH "path to GKlib")


# Add dependencies
###########################################################################################
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Insert Sources
###############################################################################################
include(${GKLIB_PATH}/GKlibSystem.cmake)
# Add include directories.
#include_directories(${GKLIB_PATH})
# Recursively look for CMakeLists.txt in subdirs.
add_subdirectory(src/include)
add_subdirectory(src/libmetis)

# Create install target
###############################################################################################
install(
	TARGETS ${PROJECT_NAME} 
	EXPORT metisTargets
    RUNTIME DESTINATION ${METIS_INSTALL_BINARY_DIR} # runtime executable
    LIBRARY DESTINATION ${METIS_INSTALL_LIB_DIR} # dynamic libraries
    ARCHIVE DESTINATION ${METIS_INSTALL_ARCHIVE_DIR} # static libraries
    PUBLIC_HEADER DESTINATION ${METIS_INSTALL_INCLUDE_DIR} # headers
)


include(CMakePackageConfigHelpers)

configure_package_config_file(
	${CMAKE_CURRENT_SOURCE_DIR}/cmake/metisConfig.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/metisConfig.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/${METIS_INSTALL_LIB_DIR}/cmake/${METIS_INSTALL_DIR}
	PATH_VARS METIS_INSTALL_INCLUDE_DIR
  )

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/metisConfigVersion.cmake
	VERSION ${METIS_VERSION}
	COMPATIBILITY AnyNewerVersion
)

export(
	TARGETS ${PROJECT_NAME} 
	NAMESPACE METIS::
	FILE ${CMAKE_CURRENT_BINARY_DIR}/metisTargets.cmake
)

install(
	EXPORT metisTargets 
	NAMESPACE METIS:: 
	DESTINATION ${CMAKE_INSTALL_PREFIX}/${METIS_INSTALL_LIB_DIR}/cmake/${METIS_INSTALL_DIR}
)

install(
	FILES
		${CMAKE_CURRENT_BINARY_DIR}/metisConfig.cmake
		${CMAKE_CURRENT_BINARY_DIR}/metisConfigVersion.cmake
	DESTINATION
		${CMAKE_INSTALL_PREFIX}/${METIS_INSTALL_LIB_DIR}/cmake/${METIS_INSTALL_DIR}
)
