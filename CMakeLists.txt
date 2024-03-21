cmake_minimum_required(VERSION 3.15)

project(
  STREAM
  VERSION 1.0
  DESCRIPTION "STREAM benchmark"
  LANGUAGES C)

enable_testing()

option(STREAM_ENABLE_CALIPER "Enable Caliper/Adiak support" FALSE)

if(CMAKE_C_COMPILER_ID MATCHES "(GNU|Clang)")
  if(CMAKE_SYSTEM_PROCESSOR MATCHES "ppc64le")
    add_compile_options("$<$<COMPILE_LANGUAGE:C>:-O3;-mcpu=native>")
  else()
    add_compile_options("$<$<COMPILE_LANGUAGE:C>:-O3;-march=native>")
  endif()
elseif(CMAKE_C_COMPILER_ID MATCHES "Intel")
  add_compile_options("$<$<COMPILE_LANGUAGE:C>:-O3;-xHost>")
endif()

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

add_definitions(-DSTREAM_CMAKE_BUILD=1)
include_directories(${CMAKE_BINARY_DIR})

configure_file(
  "${CMAKE_SOURCE_DIR}/stream-config.h.in"
  "${CMAKE_BINARY_DIR}/stream-config.h"
)

# The C STREAM benchmark only requires
# stream.c
add_executable(stream stream.c)

add_test(NAME STREAM_C COMMAND stream)

if (STREAM_ENABLE_CALIPER)
  find_package(caliper REQUIRED)
  find_package(adiak REQUIRED)
  target_link_libraries(stream PRIVATE caliper adiak::adiak)
endif()

# Look for OpenMP support is found, link it to the executables
# Note that if you are using clang on macOS, you will need to
# install libomp via Homebrew and then set the following
# environment variables:
#   export OpenMP_ROOT=$(brew --prefix)/opt/libomp
# see https://www.scivision.dev/cmake-openmp/ for more details

find_package(OpenMP COMPONENTS C)
target_link_libraries(stream PRIVATE $<$<BOOL:${OpenMP_C_FOUND}>:OpenMP::OpenMP_C>)

# Per @scivision, ignore the build directory
# (see https://www.scivision.dev/cmake-auto-gitignore-build-dir/)
if(NOT PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  file(GENERATE OUTPUT .gitignore CONTENT "*")
endif()

install(TARGETS
    stream
  DESTINATION
    ${CMAKE_INSTALL_BINDIR}
)
