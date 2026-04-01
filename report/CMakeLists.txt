cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
project(reporting VERSION 1.0 LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 14 CACHE STRING "C++ standard to build all targets.")

set(GITHUB "https://github.com/" CACHE STRING "github base url")

include(CTest)

cpmaddpackage(
  NAME SystemCLanguage
  GIT_REPOSITORY  ${GITHUB}accellera-official/systemc.git
  GIT_SHALLOW True
  GIT_TAG main
)

cpmaddpackage(
  NAME SystemCCCI
  GIT_REPOSITORY  ${GITHUB}accellera-official/cci.git
  GIT_SHALLOW True
  GIT_TAG main
)

set(WITH_FMT "true" CACHE STRING "Include FMT library")
if (WITH_FMT)
cpmaddpackage(
  NAME fmt
  GIT_REPOSITORY  ${GITHUB}fmtlib/fmt.git
  GIT_SHALLOW True
  GIT_TAG 9.1.0
  OPTIONS FMT_INSTALL "" ON
)
endif()

CPMAddPackage(
  NAME spdlog
  GIT_REPOSITORY ${GITHUB}gabime/spdlog.git
  GIT_TAG v1.9.2
  GIT_SHALLOW ON
  DOWNLOAD_ONLY YES
)

add_library(${PROJECT_NAME} src/report.cpp)

target_include_directories(
  ${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(TARGET fmt)
  target_link_libraries(${PROJECT_NAME} PUBLIC fmt::fmt)
endif()

target_include_directories(${PROJECT_NAME} PRIVATE ${spdlog_SOURCE_DIR}/include)
if(TARGET SystemC::cci)
  target_compile_definitions(${PROJECT_NAME} PRIVATE HAS_CCI)
  target_link_libraries(${PROJECT_NAME} PUBLIC SystemC::cci)
endif()
target_link_libraries(${PROJECT_NAME} PUBLIC SystemC::systemc)
set_target_properties(${PROJECT_NAME} PROPERTIES LINKER_LANGUAGE CXX)

#No tests yet. WIP.
#if(BUILD_TESTING AND ("${PROJECT_NAME}" STREQUAL "${CMAKE_PROJECT_NAME}"))
#   enable_testing()
#   add_subdirectory(tests)
#endif()

add_library("scp::report::lib${PROJECT_NAME}" ALIAS ${PROJECT_NAME})
packageproject(
   NAME "${PROJECT_NAME}"
   VERSION ${PROJECT_VERSION}
   NAMESPACE scp
   BINARY_DIR ${PROJECT_BINARY_DIR}
   INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
   INCLUDE_DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
   VERSION_HEADER "${VERSION_HEADER_LOCATION}"
   COMPATIBILITY SameMajorVersion
)
install(
   TARGETS ${PROJECT_NAME}
   LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
   COMPONENT "${PROJECT_NAME}_Runtime"
   NAMELINK_COMPONENT "${PROJECT_NAME}_Development"
)

