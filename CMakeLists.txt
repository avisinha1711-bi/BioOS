cmake_minimum_required(VERSION 3.10)
project(BioOS VERSION 1.0.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pthread")

# Enable testing
enable_testing()

# Find required packages
find_package(Threads REQUIRED)

# Set source directories
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/cpp)
set(TEST_DIR ${CMAKE_CURRENT_SOURCE_DIR}/tests)
set(INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/cpp)

# Include directories
include_directories(${INCLUDE_DIR})

# Add main executable
add_executable(bioOS_simulator
    ${SOURCE_DIR}/bioOS_kernel.cpp
    ${SOURCE_DIR}/bioOS_main.cpp
)

target_link_libraries(bioOS_simulator
    PRIVATE Threads::Threads
)

# Set output directory
set_target_properties(bioOS_simulator PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)

# Add test executable
add_executable(bioOS_tests
    ${SOURCE_DIR}/bioOS_kernel.cpp
    ${TEST_DIR}/test_bioOS_cpp.cpp
)

target_link_libraries(bioOS_tests
    PRIVATE Threads::Threads
)

set_target_properties(bioOS_tests PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)

# Register tests
add_test(NAME BioOSTests COMMAND bioOS_tests)

# Add compiler flags for different configurations
if(CMAKE_BUILD_TYPE MATCHES Debug)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -DDEBUG")
    message(STATUS "Debug build enabled")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -DNDEBUG")
    message(STATUS "Release build enabled")
endif()

# Print configuration information
message(STATUS "BioOS Configuration")
message(STATUS "===================")
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER}")
message(STATUS "Source directory: ${SOURCE_DIR}")
message(STATUS "Binary directory: ${CMAKE_BINARY_DIR}")

# Custom target for running tests
add_custom_target(run_tests
    COMMAND ${CMAKE_CTEST_COMMAND} --verbose
    DEPENDS bioOS_tests
)

# Custom target for running simulator
add_custom_target(run_simulator
    COMMAND ${CMAKE_BINARY_DIR}/bin/bioOS_simulator
    DEPENDS bioOS_simulator
)

# Custom target for coverage (if supported)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_custom_target(coverage
        COMMAND ${CMAKE_CXX_COMPILER} --version
        COMMENT "Coverage report generation"
    )
endif()

# Installation rules
install(TARGETS bioOS_simulator DESTINATION bin)
install(FILES ${INCLUDE_DIR}/bioOS_kernel.h DESTINATION include)
