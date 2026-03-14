# CTest

CTest is a testing tool that comes bundled with CMake - it's the test driver/runner for CMake projects. 

## CTest vs Google Test

### **Google Test (gtest)**
- **What it is**: A C++ testing *framework*
- **Purpose**: Provides the infrastructure to *write* tests (assertions, fixtures, parameterized tests, etc.)
- **Scope**: C++ specific
- **You use it to**: Define test cases, make assertions, organize test suites

### **CTest**
- **What it is**: A testing *runner/orchestrator*
- **Purpose**: Provides the infrastructure to *run* and *report* tests
- **Scope**: Language-agnostic (works with any test framework or executable)
- **You use it to**: Execute tests, collect results, generate reports

## The Relationship

They work **together** in a complementary way:

```
┌─────────────────────────────────────────────┐
│  You write tests using Google Test          │
│  (EXPECT_EQ, TEST_F, etc.)                  │
└─────────────────┬───────────────────────────┘
                  │
                  │ compiles to
                  ▼
┌─────────────────────────────────────────────┐
│  Test executable (e.g., my_tests)           │
└─────────────────┬───────────────────────────┘
                  │
                  │ registered with
                  ▼
┌─────────────────────────────────────────────┐
│  CTest discovers and runs the executable    │
│  (ctest command)                            │
└─────────────────────────────────────────────┘
```

## Practical Example

**With just Google Test:**
```bash
# Run your test binary directly
./my_tests

# Filter specific tests
./my_tests --gtest_filter=CalculatorTest.*
```

**With CTest:**
```bash
# Run all tests registered with CTest
ctest

# Run with verbose output
ctest -V

# Run specific test by name
ctest -R Calculator

# Run tests in parallel
ctest -j4

# Stop on first failure
ctest --stop-on-failure
```

## Key CTest Features

### 1. **Language Agnostic**
CTest can run *any* executable as a test:

```cmake
# Google Test
add_test(NAME MyGoogleTest COMMAND my_gtest_binary)

# Python test
add_test(NAME MyPythonTest COMMAND python test.py)

# Shell script
add_test(NAME MyShellTest COMMAND ./test.sh)

# Custom executable
add_test(NAME CustomTest COMMAND my_custom_test_program)
```

### 2. **Test Discovery**
CTest can automatically find Google Test cases:

```cmake
include(GoogleTest)
gtest_discover_tests(my_tests)  # Finds all TEST() macros
```

This creates individual CTest entries for each Google Test case:
```bash
$ ctest -N
Test #1: CalculatorTest.Addition
Test #2: CalculatorTest.Subtraction
Test #3: CalculatorTest.Multiplication
```

### 3. **Parallel Execution**
```bash
ctest -j8  # Run 8 tests in parallel
```

### 4. **Test Properties**
```cmake
add_test(NAME SlowTest COMMAND ./slow_test)
set_tests_properties(SlowTest PROPERTIES
    TIMEOUT 300           # 5 minute timeout
    LABELS "integration"  # Tag for grouping
    COST 100             # Run expensive tests first
)
```

### 5. **CI/CD Integration**
CTest provides structured output for CI systems:

```bash
# Dashboard submission (CDash)
ctest -D Experimental

# JUnit XML output
ctest --output-junit results.xml
```

## Complete CMake Setup Example

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject)

set(CMAKE_CXX_STANDARD 17)

# Fetch Google Test
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG v1.14.0
)
FetchContent_MakeAvailable(googletest)

# Enable CTest
enable_testing()

# Your library
add_library(mylib src/calculator.cpp)

# Test executable using Google Test
add_executable(unit_tests 
    test/test_calculator.cpp
    test/test_parser.cpp
)

target_link_libraries(unit_tests 
    mylib
    GTest::gtest_main  # Google Test framework
)

# Method 1: Auto-discover Google Test cases
include(GoogleTest)
gtest_discover_tests(unit_tests)

# Method 2: Manual registration (if you prefer control)
# add_test(NAME AllUnitTests COMMAND unit_tests)

# Integration test (plain executable, no gtest)
add_executable(integration_test test/integration_test.cpp)
target_link_libraries(integration_test mylib)
add_test(NAME IntegrationTest COMMAND integration_test)

# Set properties
set_tests_properties(IntegrationTest PROPERTIES
    TIMEOUT 60
    LABELS "integration"
)
```

## Running Tests - Comparison

### Using Google Test Binary Directly
```bash
# Pros: Direct control, gtest-specific flags
./unit_tests --gtest_filter=Calculator*
./unit_tests --gtest_repeat=100
./unit_tests --gtest_shuffle

# Cons: Must know binary names, no aggregation
```

### Using CTest
```bash
# Pros: Unified interface, parallel execution, reporting
ctest                          # Run all tests
ctest -R Calculator            # Regex filter
ctest -L integration           # Run by label
ctest -j4                      # Parallel
ctest --output-on-failure      # Show failures only

# Cons: Less gtest-specific control
```

## When to Use What?

**Use Google Test directly when:**
- Developing/debugging specific tests
- Need fine-grained gtest features (parameterized tests, death tests)
- Working on a single test suite

**Use CTest when:**
- Running full test suite in CI/CD
- Managing tests in multiple languages
- Need parallel execution
- Want unified reporting across different test types
- Integrating with CDash dashboards

## Real-World Workflow

```bash
# Development: Use gtest directly for fast iteration
$ ./build/unit_tests --gtest_filter=MyNewFeature*

# Pre-commit: Quick CTest run
$ cd build && ctest --output-on-failure

# CI/CD: Full CTest with reporting
$ ctest -j$(nproc) --output-junit results.xml
```

## Summary

| Feature | Google Test | CTest |
|---------|-------------|-------|
| **Type** | Test framework | Test runner |
| **Language** | C++ only | Any language |
| **Purpose** | Write tests | Run tests |
| **Assertions** | ✅ Rich assertion library | ❌ N/A |
| **Parallel execution** | ❌ No | ✅ Yes |
| **Test discovery** | ❌ No | ✅ Yes |
| **CI integration** | Limited | ✅ Excellent |
| **Cross-framework** | ❌ No | ✅ Yes |

**Bottom line**: Google Test helps you *write* C++ tests, CTest helps you *run* them (along with any other tests). They're complementary tools that work great together!