cmake_minimum_required(VERSION 3.14)
project(MyProject)

# GoogleTest requires at least C++14
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Fetch GoogleTest
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        v1.14.0
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

# Enable testing
enable_testing()

# Add your executable
add_executable(my_tests test_calculator.cpp)

# Link against gtest
target_link_libraries(my_tests GTest::gtest_main)

# Discover tests
include(GoogleTest)
gtest_discover_tests(my_tests)