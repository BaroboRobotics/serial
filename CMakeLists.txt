cmake_minimum_required(VERSION 2.8.3)
project(serial)

option(BUILD_TESTS "Build unit tests. Requires gtest" OFF)

if(APPLE)
	find_library(IOKIT_LIBRARY IOKit)
	find_library(FOUNDATION_LIBRARY Foundation)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(APPLE)
	# If OSX
	list(APPEND serial_SRCS src/impl/unix.cc)
	list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(APPLE)
	target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
	target_link_libraries(${PROJECT_NAME} rt pthread)
else()
	target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
add_dependencies(serial_example ${PROJECT_NAME})
target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

if(UNIX)
  set(INSTALL_LIB_DIR lib)
  set(INSTALL_BIN_DIR bin)
else()
  set(INSTALL_LIB_DIR .)
  set(INSTALL_BIN_DIR .)
endif()

set(install_args)
if(NOT BUILD_SHARED_LIBS)
  list(APPEND install_args ARCHIVE DESTINATION ${INSTALL_LIB_DIR})
endif()
## Install executable
install(TARGETS ${PROJECT_NAME}
    ${install_args}
    LIBRARY DESTINATION ${INSTALL_LIB_DIR}
)

if(BUILD_TESTS)
    if(NOT DEFINED ENV{GTEST_DIR})
        message(FATAL_ERROR "BUILD_TESTS set, but GTEST_DIR envvar not found!")
    endif()

    add_subdirectory(tests)
endif()
## Install headers
#install(FILES include/serial/serial.h include/serial/v8stdint.h
#  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)

## Tests
#if(CATKIN_ENABLE_TESTING)
#endif()
