# Copyright Louis Dionne 2013-2017
# Distributed under the Boost Software License, Version 1.0.
# (See accompanying file LICENSE.md or copy at http://boost.org/LICENSE_1_0.txt)

cmake_minimum_required(VERSION 3.9)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

##############################################################################
# Setup CMake options
##############################################################################
option(BOOST_HANA_ENABLE_CONCEPT_CHECKS "Enable concept checking in the interface methods." ON)
option(BOOST_HANA_ENABLE_DEBUG_MODE "Enable Hana's debug mode." OFF)

option(BOOST_HANA_ENABLE_STRING_UDL
"Enable the GNU extension allowing the special string literal operator\
 template, which enables the _s suffix for creating compile-time strings." ON)

option(BOOST_HANA_ENABLE_EXCEPTIONS
"Build with exceptions enabled. Note that Hana does not make use of exceptions,\
 but this switch can be disabled when building the tests to assess that it is\
 really the case." ON)


##############################################################################
# Setup project
#
# We parse the canonical version number located in <boost/hana/version.hpp>.
# This is done to allow the library to be used without requiring a proper
# installation during which the version would be written to this header.
##############################################################################
foreach(level MAJOR MINOR PATCH)
    file(STRINGS include/boost/hana/version.hpp
                 _define_${level}
                 REGEX "#define BOOST_HANA_${level}_VERSION")
    string(REGEX MATCH "([0-9]+)" _version_${level} "${_define_${level}}")
endforeach()

set(Boost.Hana_VERSION_STRING "${_version_MAJOR}.${_version_MINOR}.${_version_PATCH}")
project(Boost.Hana VERSION ${Boost.Hana_VERSION_STRING} LANGUAGES CXX)

# Perform checks to make sure we support the current compiler
include(CheckCxxCompilerSupport)


##############################################################################
# Setup the 'hana' header-only library target, along with its install target
# and exports.
##############################################################################
include(CheckCXXCompilerFlag)
add_library(hana INTERFACE)
target_include_directories(hana INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>")
target_compile_features(hana INTERFACE cxx_std_14)

# Export the `hana` library into a HanaConfig.cmake file
install(TARGETS hana
        EXPORT HanaConfig
        INCLUDES DESTINATION include)
install(EXPORT HanaConfig
        DESTINATION lib/cmake/hana)
install(DIRECTORY include/boost
        DESTINATION include
        FILES_MATCHING PATTERN "*.hpp")

# Also install an optional pkg-config file
configure_file(cmake/hana.pc.in hana.pc @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/hana.pc" DESTINATION lib/pkgconfig)
