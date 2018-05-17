list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

cmake_minimum_required(VERSION 2.8)

project(ALGLIB CXX)

set(libraryname ALGLIB)

include(AddWarningsConfigurationToTargets)

include(CMakePackageConfigHelpers)

include(GNUInstallDirs)

option(ENABLE_RPATH "Enable RPATH for this library" ON)
mark_as_advanced(ENABLE_RPATH)
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}"
                          LIB_DIRS "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}"
                          DEPENDS ENABLE_RPATH
                          USE_LINK_PATH)

include(ExternalProject)

set(ALGLIB_VERSION_FILE "3.13.0" CACHE STRING "Version of ALGLIB.")
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
set_property(TARGET ${libraryname} PROPERTY PUBLIC_HEADER ${ALGLIB_HEADERS})

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
                              ${CMAKE_BINARY_DIR}/ALGLIB-config.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ALGLIB)

install(FILES ${CMAKE_BINARY_DIR}/ALGLIB-config.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ALGLIB)

include(AddUninstallTarget)

