list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

cmake_minimum_required(VERSION 2.8)

project(alglib CXX)

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

set(ALGLIB_VERSION "3.13.0")
set(ALGLIB_FILE alglib-${ALGLIB_VERSION}.cpp.gpl.tgz)
set(ALGLIB_URI http://www.alglib.net/translator/re/${ALGLIB_FILE})
set(ALGLIB_DOWNLOAD_DIR ${CMAKE_CURRENT_BINARY_DIR}/alglibSource)
set(ALGLIB_FULL_PATH ${ALGLIB_DOWNLOAD_DIR}/${ALGLIB_FILE})

if (NOT EXISTS ${ALGLIB_FULL_PATH})
    file(DOWNLOAD ${ALGLIB_URI} ${ALGLIB_FULL_PATH} SHOW_PROGRESS STATUS DL_STATUS)
    list(GET DL_STATUS 0 status_code)
    if (NOT status_code EQUAL 0)
        message(FATAL_ERROR "Error while downloading '${ALGLIB_URI}'.")
    endif()
endif()

file(REMOVE_RECURSE "${ALGLIB_DOWNLOAD_DIR}/src")
file(MAKE_DIRECTORY ${ALGLIB_DOWNLOAD_DIR}/src)
execute_process(COMMAND ${CMAKE_COMMAND} -E tar xfz ${ALGLIB_FULL_PATH} WORKING_DIRECTORY "${ALGLIB_DOWNLOAD_DIR}/src" RESULT_VARIABLE rv)

if(NOT rv EQUAL 0)
  message(FATAL_ERROR "Error while extracting '${ALGLIB_FULL_PATH}' failed")
endif()

file(COPY src-cmake/CMakeLists.txt DESTINATION ${ALGLIB_DOWNLOAD_DIR})
file(COPY cmake/AddInstallRPATHSupport.cmake cmake/AddUninstallTarget.cmake cmake/AddWarningsConfigurationToTargets.cmake DESTINATION ${ALGLIB_DOWNLOAD_DIR}/cmake)

file(MAKE_DIRECTORY ${ALGLIB_DOWNLOAD_DIR}/build)

SET_DIRECTORY_PROPERTIES(PROPERTIES EP_PREFIX ${CMAKE_CURRENT_BINARY_DIR})


ExternalProject_Add(alglib
                    SOURCE_DIR ${ALGLIB_DOWNLOAD_DIR}
                    BINARY_DIR ${ALGLIB_DOWNLOAD_DIR}/build
                    CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}")

include(AddUninstallTarget)


#devo ancora settare il find package e le impostazioni aggiuntive specificate nel readme
