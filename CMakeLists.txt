# Copyright (C) 2018 Fondazione Istituto Italiano di Tecnologia
#
# Licensed under either the GNU Lesser General Public License v3.0 :
# https://www.gnu.org/licenses/lgpl-3.0.html
# or the GNU Lesser General Public License v2.1 :
# https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html
# at your option.

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

cmake_minimum_required(VERSION 2.8)

project(ALGLIB CXX)

set(libraryname ALGLIB)

include(AddWarningsConfigurationToTargets)

include(CMakePackageConfigHelpers)

include(GNUInstallDirs)

# ouptut paths
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

# Build shared libs
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

if(MSVC)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()

option(BUILD_SHARED_LIBS "Build libraries as shared as opposed to static" ON)

option(ENABLE_RPATH "Enable RPATH for this library" ON)
mark_as_advanced(ENABLE_RPATH)
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}"
                          LIB_DIRS "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}"
                          DEPENDS ENABLE_RPATH
                          USE_LINK_PATH)

# Encourage user to specify a build type (e.g. Release, Debug, etc.), otherwise set it to Release.
if(NOT CMAKE_CONFIGURATION_TYPES)
  if(NOT CMAKE_BUILD_TYPE)
      message(STATUS "Setting build type to 'Release' as none was specified.")
      set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE "Release")
  endif()
endif()

include(ExternalProject)

set(ALGLIB_VERSION_FILE "3.16.0" CACHE STRING "Version of ALGLIB.")
string(REPLACE "\"" "" ALGLIB_VERSION ${ALGLIB_VERSION_FILE})
set(ALGLIB_FILE alglib-${ALGLIB_VERSION_FILE}.cpp.gpl.tgz)
set(ALGLIB_URI http://www.alglib.net/translator/re/${ALGLIB_FILE})
set(ALGLIB_DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(ALGLIB_FULL_PATH ${ALGLIB_DOWNLOAD_DIR}/${ALGLIB_FILE})

if (NOT EXISTS ${ALGLIB_FULL_PATH})
    file(DOWNLOAD ${ALGLIB_URI} ${ALGLIB_FULL_PATH} SHOW_PROGRESS STATUS DL_STATUS)
    list(GET DL_STATUS 0 status_code)
    if (NOT status_code EQUAL 0)
        message(FATAL_ERROR "Error while downloading '${ALGLIB_URI}'.")
    endif()
endif()

set (UNZIP_DIR "${ALGLIB_DOWNLOAD_DIR}/src")
file(REMOVE_RECURSE ${UNZIP_DIR})
file(MAKE_DIRECTORY ${UNZIP_DIR})
execute_process(COMMAND ${CMAKE_COMMAND} -E tar xfz ${ALGLIB_FULL_PATH} WORKING_DIRECTORY "${ALGLIB_DOWNLOAD_DIR}/src" RESULT_VARIABLE rv)

if(NOT rv EQUAL 0)
  message(FATAL_ERROR "Error while extracting '${ALGLIB_FULL_PATH}'.")
endif()

file(GLOB ALGLIB_HEADERS "${UNZIP_DIR}/cpp/src/*.h")
file(GLOB ALGLIB_SOURCES "${UNZIP_DIR}/cpp/src/*.cpp")

add_library(${libraryname} ${ALGLIB_HEADERS} ${ALGLIB_SOURCES})
set_property(TARGET ${libraryname} PROPERTY POSITION_INDEPENDENT_CODE ON)

set(COMPILER_OPTIONS)

if(MSVC OR MSYS OR MINGW)
    set(COMPILER_OPTIONS ${COMPILER_OPTIONS} /Ox /DAE_OS=AE_WINDOWS)
    execute_process( COMMAND wmic CPU get Name OUTPUT_VARIABLE CPU_NAME)
    if (${CPU_NAME} MATCHES "Intel")
        set(COMPILER_OPTIONS ${COMPILER_OPTIONS} /DAE_CPU=AE_INTEL)
    endif()

elseif(UNIX)
    set(COMPILER_OPTIONS ${COMPILER_OPTIONS} -O3 -DAE_OS=AE_POSIX -pthread)
    execute_process( COMMAND uname -m OUTPUT_VARIABLE ARCHITECTURE)

    if (${ARCHITECTURE} MATCHES "x86_64")
        set(COMPILER_OPTIONS ${COMPILER_OPTIONS} -DAE_CPU=AE_INTEL)
    endif()
endif()


target_compile_options(${libraryname} PUBLIC ${COMPILER_OPTIONS})

# install
set_target_properties(${libraryname} PROPERTIES VERSION       ${ALGLIB_VERSION}
                                                PUBLIC_HEADER "${ALGLIB_HEADERS}")

target_include_directories(${libraryname} PUBLIC "$<BUILD_INTERFACE:${UNZIP_DIR}/cpp/src>"
                                                 "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>")

install(TARGETS ${libraryname}
        EXPORT ${libraryname}
        COMPONENT runtime
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT bin
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT shlib
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT lib
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT ${libraryname}
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${libraryname})

configure_package_config_file(${CMAKE_SOURCE_DIR}/cmake/ALGLIB-config.cmake.in
                              ${CMAKE_BINARY_DIR}/alglib-config.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ALGLIB)

write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/alglib-configVersion.cmake VERSION ${ALGLIB_VERSION} COMPATIBILITY ExactVersion)

install(FILES ${CMAKE_BINARY_DIR}/alglib-config.cmake ${CMAKE_BINARY_DIR}/alglib-configVersion.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ALGLIB)

include(AddUninstallTarget)

