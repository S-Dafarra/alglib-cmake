list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

cmake_minimum_required(VERSION 2.8)

project(alglibSource CXX)

include(AddWarningsConfigurationToTargets)

include(CMakePackageConfigHelpers)

include(GNUInstallDirs)

set(libraryname alglib)

option(ENABLE_RPATH "Enable RPATH for this library" ON)
mark_as_advanced(ENABLE_RPATH)
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_BINDIR}"
                          LIB_DIRS "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}"
                          DEPENDS ENABLE_RPATH
                          USE_LINK_PATH)

file(GLOB ALGLIB_HEADERS "src/cpp/src/*.h")
file(GLOB ALGLIB_SOURCES "src/cpp/src/*.cpp")

add_library(${libraryname} ${ALGLIB_HEADERS} ${ALGLIB_SOURCES})

# install
set_property(TARGET ${libraryname} PROPERTY PUBLIC_HEADER ${ALGLIB_HEADERS})

install(TARGETS ${libraryname}
        EXPORT ${libraryname}
        COMPONENT runtime
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT bin
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT shlib
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT lib
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

include(AddUninstallTarget)



