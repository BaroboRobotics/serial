if(UNIX)
    message(STATUS "GTEST_DIR: $ENV{GTEST_DIR}")
    add_subdirectory(
        $ENV{GTEST_DIR}
        ${PROJECT_BINARY_DIR}/gtest
        )
    include_directories(${gtest_SOURCE_DIR}/include)
    link_directories(${gtest_BINARY_DIR})
    add_executable(${PROJECT_NAME}-test 
        unix_serial_tests.cc
        ${gtest_SOURCE_DIR}/include/gtest/gtest.h)
    target_link_libraries(${PROJECT_NAME}-test 
        ${PROJECT_NAME} 
        ${Boost_LIBRARIES}
        gtest)
    if(NOT APPLE)
        target_link_libraries(${PROJECT_NAME}-test util)
    endif()

    add_executable(${PROJECT_NAME}-test-timer unit/unix_timer_tests.cc)
    target_link_libraries(${PROJECT_NAME}-test-timer 
        ${PROJECT_NAME}
        gtest
        )

    enable_testing()
    add_test(${PROJECT_NAME}-test ${PROJECT_NAME}-test )
    add_test(${PROJECT_NAME}-test-timer ${PROJECT_NAME}-test-timer)
endif()
