# Copyright 2018, 2019 Peter Dimov
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt

#======================================================================
# 1. Enable experimental 'import std;' support
#======================================================================
# gersemi: off
if(NOT DEFINED CMAKE_EXPERIMENTAL_CXX_IMPORT_STD)
    if(CMAKE_VERSION VERSION_GREATER_EQUAL 4.3 AND CMAKE_VERSION VERSION_LESS 4.4)
        set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "451f2fe2-a8a2-47c3-bc32-94786d8fc91b")
    elseif(CMAKE_VERSION VERSION_GREATER_EQUAL 4.2 AND CMAKE_VERSION VERSION_LESS 4.3)
        set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "d0edc3af-4c50-42ea-a356-e2862fe7a444")
    elseif(CMAKE_VERSION VERSION_GREATER 4.3)
        message( WARNING "This version of CMake has not been tested for 'import std;'.")
    endif()
endif()
# gersemi: on

cmake_minimum_required(VERSION 3.21...4.3)

project(cmake_subdir_test LANGUAGES CXX VERSION 1.90.0)

add_subdirectory(../.. boostorg/charconv)

set(deps
    # Primary dependencies
    assert
    config
    core
    # Secondary dependencies
    throw_exception
)

foreach(dep IN LISTS deps)
    add_subdirectory(../../../${dep} boostorg/${dep})
endforeach()

include(cmake/boost-install-library.cmake)
boost_install_library(boost_core TARGETS boost_core)
boost_install_library(boost_config TARGETS boost_config)
boost_install_library(boost_assert TARGETS boost_assert)
boost_install_library(boost_throw_exception TARGETS boost_throw_exception DEPENDENCIES Boost::assert)
boost_install_library(boost_charconv TARGETS boost_charconv DEPENDENCIES Boost::assert Boost::core Boost::config Boost::throw_exception)

add_executable(quick ../quick.cpp)
target_link_libraries(quick Boost::charconv)

enable_testing()
add_test(quick quick)

add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure -C $<CONFIG>)
