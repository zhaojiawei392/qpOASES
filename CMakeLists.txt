##
##     This file is part of qpOASES.
##
##     qpOASES -- An Implementation of the Online Active Set Strategy.
##     Copyright (C) 2007-2017 by Hans Joachim Ferreau, Andreas Potschka,
##     Christian Kirches et al. All rights reserved.
##
##     qpOASES is free software; you can redistribute it and/or
##     modify it under the terms of the GNU Lesser General Public
##     License as published by the Free Software Foundation; either
##     version 2.1 of the License, or (at your option) any later version.
##
##     qpOASES is distributed in the hope that it will be useful,
##     but WITHOUT ANY WARRANTY; without even the implied warranty of
##     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
##     See the GNU Lesser General Public License for more details.
##
##     You should have received a copy of the GNU Lesser General Public
##     License along with qpOASES; if not, write to the Free Software
##     Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
##



##
##     Filename:  CMakeLists.txt
##     Author:    Hans Joachim Ferreau (thanks to Milan Vukov)
##     Version:   3.2
##     Date:      2007-2017
##

cmake_minimum_required(VERSION 3.15)

# Use modern project() syntax
project(qpOASES
    VERSION 3.2.2
    DESCRIPTION "An implementation of the online active set strategy"
    HOMEPAGE_URL "https://projects.coin-or.org/qpOASES"
    LANGUAGES CXX
)

# Use modern variable names
set(qpOASES_VERSION_MAJOR 3)
set(qpOASES_VERSION_MINOR 2)
set(qpOASES_VERSION_PATCH 2)
set(qpOASES_SOVERSION "${qpOASES_VERSION_MAJOR}.${qpOASES_VERSION_MINOR}")

# Use GNUInstallDirs for standard installation directories
include(GNUInstallDirs)

# Define build options using modern syntax
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(QPOASES_BUILD_EXAMPLES "Build examples" OFF)
option(QPOASES_AVOID_LA_NAMING_CONFLICTS "Avoid Blas/Lapack naming conflicts" OFF)

# Prevent shared library build on Windows
if(BUILD_SHARED_LIBS AND WIN32)
    message(FATAL_ERROR "Shared library build is not supported on Windows")
endif()

# Create main library target
add_library(qpOASES)
add_library(qpOASES::qpOASES ALIAS qpOASES)

# Set modern target properties
target_compile_definitions(qpOASES
    PRIVATE
        __NO_COPYRIGHT__
        $<$<BOOL:${QPOASES_AVOID_LA_NAMING_CONFLICTS}>:__AVOID_LA_NAMING_CONFLICTS__>
        $<$<CONFIG:Debug>:__DEBUG__>
        $<$<PLATFORM_ID:Linux>:LINUX>
        $<$<PLATFORM_ID:Windows>:WIN32>
)

# Set compiler flags using modern target_compile_options
if(UNIX)
    target_compile_options(qpOASES PRIVATE
        -Wall
        -pedantic
        -Wfloat-equal
        -Wshadow
        $<$<CONFIG:Release>:-O3 -finline-functions>
    )
elseif(WIN32)
    target_compile_options(qpOASES PRIVATE
        -nologo
        -EHsc
    )
endif()

# Add source files
file(GLOB SOURCES "src/*.cpp")
target_sources(qpOASES
    PRIVATE
        ${SOURCES}
)

# Set include directories
target_include_directories(qpOASES
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Set library properties
set_target_properties(qpOASES PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${qpOASES_SOVERSION}
    EXPORT_NAME qpOASES
)

# Install targets
install(TARGETS qpOASES
    EXPORT qpOASESTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

export(EXPORT qpOASESTargets 
        FILE ${CMAKE_CURRENT_BINARY_DIR}/qpOASESTargets.cmake
)

# Install headers
install(FILES include/qpOASES.hpp
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY include/qpOASES
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING 
        PATTERN "*.hpp"
        PATTERN "*.ipp"
    PATTERN ".svn" EXCLUDE
)

# Export targets
install(EXPORT qpOASESTargets
    FILE qpOASESTargets.cmake
    NAMESPACE qpOASES::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/qpOASES
)

# Generate and install package configuration files
include(CMakePackageConfigHelpers)

configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/qpOASESConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/qpOASESConfig.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/qpOASES
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/qpOASESConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/qpOASESConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/qpOASESConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/qpOASES
)

# Examples
if(QPOASES_BUILD_EXAMPLES)
    set(EXAMPLE_NAMES
        example1
        example1a
        example1b
        example2
        example3
        example3b
        example4
        example5
        exampleLP
        qrecipe
        qrecipeSchur
    )

    foreach(example ${EXAMPLE_NAMES})
        add_executable(${example} examples/${example}.cpp)
        target_link_libraries(${example} PRIVATE qpOASES::qpOASES)
    endforeach()
endif()

##
##   end of file
##
