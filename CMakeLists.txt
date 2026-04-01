cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
project(scp VERSION 1.0 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 14 CACHE STRING "C++ standard to build all targets.")

set(GITHUB "https://github.com/" CACHE STRING "github base url")

include(CTest)

file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.42.1/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH SHA256=f3a6dcc6a04ce9e7f51a127307fa4f699fb2bade357a8eb4c5b45df76e1dc6a5
)
include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)

cpmaddpackage("${GITHUB}TheLartians/PackageProject.cmake.git@1.4.1")

cpmaddpackage(
  NAME SystemCLanguage
  GIT_REPOSITORY  ${GITHUB}accellera-official/systemc.git
  GIT_SHALLOW True
  GIT_TAG main
)

cpmaddpackage(
  NAME initiator_id
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/tlm_extensions/initiator_id
)
cpmaddpackage(
  NAME path_trace
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/tlm_extensions/path_trace
)
cpmaddpackage(
  NAME report
  SOURCE_DIR ${PROJECT_SOURCE_DIR}/report
)

if("${PROJECT_NAME}" STREQUAL "${CMAKE_PROJECT_NAME}")
  add_subdirectory(examples)
  if(BUILD_TESTING)
    enable_testing()
  endif()
endif()
