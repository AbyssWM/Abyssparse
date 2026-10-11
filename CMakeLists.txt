cmake_minimum_required(VERSION 3.19)

file(READ "${CMAKE_SOURCE_DIR}/VERSION" VER_RAW)
string(STRIP ${VER_RAW} ABYSSPARSE_VERSION)

project(
  abyssparse
  VERSION ${ABYSSPARSE_VERSION}
  DESCRIPTION "A library to parse abyss config files")

include(CTest)
include(GNUInstallDirs)

set(PREFIX ${CMAKE_INSTALL_PREFIX})
set(INCLUDE ${CMAKE_INSTALL_FULL_INCLUDEDIR})
set(LIBDIR ${CMAKE_INSTALL_FULL_LIBDIR})

configure_file(abyssparse.pc.in abyssparse.pc @ONLY)

if(CMAKE_BUILD_TYPE MATCHES Debug OR CMAKE_BUILD_TYPE MATCHES DEBUG)
  message(STATUS "Configuring abyssparse in Debug")
  add_compile_definitions(ABYSSLAND_DEBUG)
else()
  add_compile_options(-O3)
  message(STATUS "Configuring abyssparse in Release")
endif()

add_compile_definitions(ABYSSPARSE_INTERNAL)

set(CMAKE_CXX_STANDARD 23)
add_compile_options(
  -Wall
  -Wextra
  -Wpedantic
  -Wno-unused-parameter
  -Wno-missing-field-initializers)
set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)

find_package(PkgConfig REQUIRED)
pkg_check_modules(deps REQUIRED IMPORTED_TARGET abyssutils>=0.7.1)

file(GLOB_RECURSE SRCFILES CONFIGURE_DEPENDS "src/*.cpp" "include/abyssparse.hpp")

add_library(abyssparse SHARED ${SRCFILES})
target_include_directories(
  abyssparse
  PUBLIC "./include"
  PRIVATE "./src")
set_target_properties(
  abyssparse
  PROPERTIES VERSION ${ABYSSPARSE_VERSION}
             SOVERSION 2
             PUBLIC_HEADER include/abyssparse.hpp)

target_link_libraries(abyssparse PkgConfig::deps)

add_library(abyss::abyssparse ALIAS abyssparse)
install(TARGETS abyssparse)

# tests
add_custom_target(tests)

add_executable(abyssparse_test "tests/parse/main.cpp")
target_link_libraries(abyssparse_test PRIVATE abyss::abyssparse)
add_test(
  NAME "Parsing"
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/tests
  COMMAND abyssparse_test "parse")
add_dependencies(tests abyssparse_test)

add_executable(abyssparse_fuzz "tests/fuzz/main.cpp")
target_link_libraries(abyssparse_fuzz PRIVATE abyss::abyssparse)
add_test(
  NAME "Fuzz"
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/tests
  COMMAND abyssparse_fuzz "fuzz")
add_dependencies(tests abyssparse_fuzz)

# Installation
install(
  TARGETS abyssparse
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES ${CMAKE_BINARY_DIR}/abyssparse.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
