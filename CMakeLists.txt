cmake_minimum_required(VERSION 3.0.0)
project(rpc_catkin)
find_package(catkin REQUIRED)


set(PTHREAD_LIB "")
find_package(Threads)
if(CMAKE_THREAD_LIBS_INIT)
  string(LENGTH ${CMAKE_THREAD_LIBS_INIT} _length)
  if(_length GREATER 2)
    string(SUBSTRING ${CMAKE_THREAD_LIBS_INIT} 0 2 _prefix)
    if(${_prefix} STREQUAL "-l")
      math(EXPR _rest_len "${_length} - 2")
      string(SUBSTRING ${CMAKE_THREAD_LIBS_INIT} 2 ${_rest_len} PTHREAD_LIB)
    endif()
  endif()
endif()


catkin_package(
   INCLUDE_DIRS include
   LIBRARIES ${PROJECT_NAME} ${PTHREAD_LIB}
)



set(RPCLIB_VERSION_MAJOR 2)
set(RPCLIB_VERSION_MINOR 1)
set(RPCLIB_VERSION_PATCH 0)

set(RPCLIB_DEFAULT_PORT 8080 CACHE STRING "Default port used for running tests and examples")
set(RPCLIB_DEFAULT_BUFFER_SIZE "1024 << 10" CACHE STRING "Default buffer size")
set(RPCLIB_CXX_STANDARD 11 CACHE STRING "C++ version used to build rpclib (Currently: Only 11 and 14 supported)")
set(CMAKE_CXX_STANDARD 11)

set(RPCLIB_ARCH_DEF "RPCLIB_ARCH_X64")
set(RPCLIB_TARGET_ARCH "x64")
set(RPCLIB_DEB_ARCH "amd64")

set(RPCLIB_COMPILE_DEFINITIONS
    "${RPCLIB_COMPILE_DEFINITIONS}"
    "${RPCLIB_ARCH_DEF}"
    "${RPCLIB_OS_DEF}"
    "ASIO_STANDALONE"
    "RPCLIB_ASIO=clmdep_asio"
    "RPCLIB_FMT=clmdep_fmt"
    "RPCLIB_MSGPACK=clmdep_msgpack"
    )

configure_file(
    "${PROJECT_SOURCE_DIR}/include/rpc/version.h.in"
    "${PROJECT_SOURCE_DIR}/include/rpc/version.h")

configure_file(
    "${PROJECT_SOURCE_DIR}/include/rpc/config.h.in"
    "${PROJECT_SOURCE_DIR}/include/rpc/config.h")


file(GLOB_RECURSE RPCLIB_HEADERS
    include/rpc/*.h
    include/msgpack/*.hpp)
file(GLOB_RECURSE DEP_HEADERS
    dependencies/include/*.h
    dependencies/include/*.hpp)

set(DEP_SOURCES
    dependencies/src/format.cc
    dependencies/src/posix.cc)

include_directories(
 include
 include/rpc
 dependencies/include
 ${catkin_INCLUDE_DIRS}
)


add_library(${PROJECT_NAME} STATIC
    lib/rpc/dispatcher.cc
    lib/rpc/server.cc
    lib/rpc/client.cc
    lib/rpc/this_handler.cc
    lib/rpc/this_session.cc
    lib/rpc/this_server.cc
    lib/rpc/rpc_error.cc
    lib/rpc/detail/server_session.cc
    lib/rpc/detail/response.cc
    lib/rpc/detail/client_error.cc
    lib/rpc/nonstd/optional.cc
    ${DEP_SOURCES}
    ${DEP_HEADERS}
    ${RPCLIB_HEADERS})


set_target_properties(${PROJECT_NAME}
        PROPERTIES
        COMPILE_DEFINITIONS "${RPCLIB_COMPILE_DEFINITIONS}")


target_link_libraries(${PROJECT_NAME})
#get_cmake_property(_variableNames VARIABLES)
#foreach (_variableName ${_variableNames})
#  message(STATUS "${_variableName}=${${_variableName}}")
#endforeach()
