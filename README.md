# Complete Google Test Tutorial for C++

## Table of Contents
1. [Introduction](#introduction)
2. [Installation & Setup](#installation--setup)
3. [Basic Concepts](#basic-concepts)
4. [Writing Your First Test](#writing-your-first-test)
5. [Assertions](#assertions)
6. [Test Fixtures](#test-fixtures)
7. [Advanced Features](#advanced-features)
8. [Best Practices](#best-practices)
9. [Real-World Examples](#real-world-examples)

---

## Introduction

Google Test (also known as gtest) is a C++ testing framework developed by Google. It's widely used in industry for unit testing, integration testing, and more.

### Why Google Test?

- **Cross-platform**: Works on Linux, macOS, Windows
- **Rich assertion library**: Comprehensive set of macros for testing
- **Test discovery**: Automatically finds and runs tests
- **Death tests**: Test code that should crash
- **Value-parameterized tests**: Run same test with different inputs
- **Type-parameterized tests**: Test generic code
- **Integration with CI/CD**: Easy to integrate with build systems

---

## Installation & Setup

### Method 1: Using CMake (Recommended)

**Step 1: Add Google Test to your project**

Create a `CMakeLists.txt`:

```cmake
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
```

**Step 2: Build and run**

```bash
mkdir build
cd build
cmake ..
cmake --build .
ctest
```

### Method 2: Manual Installation

```bash
# Clone the repository
git clone https://github.com/google/googletest.git
cd googletest

# Build and install
mkdir build
cd build
cmake ..
make
sudo make install
```

Then in your `CMakeLists.txt`:

```cmake
find_package(GTest REQUIRED)
target_link_libraries(my_tests GTest::GTest GTest::Main)
```

---

## Basic Concepts

### Test Structure

Google Test organizes tests into **test suites** (formerly called test cases) and **test cases**:

```cpp
TEST(TestSuiteName, TestName) {
    // Test body
}
```

- **Test Suite**: Groups related tests (e.g., `MathOperations`)
- **Test Name**: Describes what's being tested (e.g., `AdditionWorks`)

### Assertions

Two types of assertions:

1. **ASSERT_***: Fatal - stops current function on failure
2. **EXPECT_***: Non-fatal - continues after failure

```cpp
EXPECT_EQ(2 + 2, 4);  // Non-fatal: continues if fails
ASSERT_EQ(2 + 2, 4);  // Fatal: stops test if fails
```

---

## Writing Your First Test

### Example: Testing a Calculator

**calculator.h**
```cpp
#ifndef CALCULATOR_H
#define CALCULATOR_H

class Calculator {
public:
    int add(int a, int b) { return a + b; }
    int subtract(int a, int b) { return a - b; }
    int multiply(int a, int b) { return a * b; }
    double divide(int a, int b) {
        if (b == 0) throw std::invalid_argument("Division by zero");
        return static_cast<double>(a) / b;
    }
};

#endif
```

**test_calculator.cpp**
```cpp
#include <gtest/gtest.h>
#include "calculator.h"

TEST(CalculatorTest, Addition) {
    Calculator calc;
    EXPECT_EQ(calc.add(2, 3), 5);
    EXPECT_EQ(calc.add(-1, 1), 0);
    EXPECT_EQ(calc.add(0, 0), 0);
}

TEST(CalculatorTest, Subtraction) {
    Calculator calc;
    EXPECT_EQ(calc.subtract(5, 3), 2);
    EXPECT_EQ(calc.subtract(0, 5), -5);
}

TEST(CalculatorTest, Multiplication) {
    Calculator calc;
    EXPECT_EQ(calc.multiply(3, 4), 12);
    EXPECT_EQ(calc.multiply(-2, 3), -6);
}

TEST(CalculatorTest, Division) {
    Calculator calc;
    EXPECT_DOUBLE_EQ(calc.divide(10, 2), 5.0);
    EXPECT_DOUBLE_EQ(calc.divide(7, 2), 3.5);
}

TEST(CalculatorTest, DivisionByZero) {
    Calculator calc;
    EXPECT_THROW(calc.divide(5, 0), std::invalid_argument);
}
```

---

## Assertions

### Basic Assertions

```cpp
// Boolean conditions
EXPECT_TRUE(condition);
EXPECT_FALSE(condition);

// Equality
EXPECT_EQ(val1, val2);    // val1 == val2
EXPECT_NE(val1, val2);    // val1 != val2

// Comparison
EXPECT_LT(val1, val2);    // val1 < val2
EXPECT_LE(val1, val2);    // val1 <= val2
EXPECT_GT(val1, val2);    // val1 > val2
EXPECT_GE(val1, val2);    // val1 >= val2
```

### String Assertions

```cpp
EXPECT_STREQ(str1, str2);     // C strings equal
EXPECT_STRNE(str1, str2);     // C strings not equal
EXPECT_STRCASEEQ(str1, str2); // Case-insensitive equality
```

### Floating Point Assertions

```cpp
EXPECT_FLOAT_EQ(val1, val2);   // Float equality (4 ULPs)
EXPECT_DOUBLE_EQ(val1, val2);  // Double equality (4 ULPs)
EXPECT_NEAR(val1, val2, abs_error);  // Within absolute error
```

### Exception Assertions

```cpp
EXPECT_THROW(statement, exception_type);  // Throws specific exception
EXPECT_ANY_THROW(statement);              // Throws any exception
EXPECT_NO_THROW(statement);               // Doesn't throw
```

### Custom Messages

```cpp
EXPECT_EQ(x, y) << "x and y should be equal, but x=" << x << " and y=" << y;
```

---

## Test Fixtures

Test fixtures allow you to reuse setup/teardown code across multiple tests.

### Basic Fixture

```cpp
#include <gtest/gtest.h>
#include <vector>

class VectorTest : public ::testing::Test {
protected:
    // Per-test setup
    void SetUp() override {
        vec.push_back(1);
        vec.push_back(2);
        vec.push_back(3);
    }

    // Per-test teardown
    void TearDown() override {
        vec.clear();
    }

    // Shared resources
    std::vector<int> vec;
};

TEST_F(VectorTest, SizeTest) {
    EXPECT_EQ(vec.size(), 3);
}

TEST_F(VectorTest, ElementAccess) {
    EXPECT_EQ(vec[0], 1);
    EXPECT_EQ(vec[1], 2);
    EXPECT_EQ(vec[2], 3);
}

TEST_F(VectorTest, PushBack) {
    vec.push_back(4);
    EXPECT_EQ(vec.size(), 4);
    EXPECT_EQ(vec[3], 4);
}
```

### Shared Setup for All Tests

```cpp
class DatabaseTest : public ::testing::Test {
protected:
    // Called once before all tests
    static void SetUpTestSuite() {
        database = new Database();
        database->connect("test_db");
    }

    // Called once after all tests
    static void TearDownTestSuite() {
        database->disconnect();
        delete database;
        database = nullptr;
    }

    static Database* database;
};

Database* DatabaseTest::database = nullptr;

TEST_F(DatabaseTest, InsertRecord) {
    EXPECT_TRUE(database->insert("key", "value"));
}

TEST_F(DatabaseTest, RetrieveRecord) {
    database->insert("key", "value");
    EXPECT_EQ(database->get("key"), "value");
}
```

---

## Advanced Features

### Parameterized Tests

Run the same test with different inputs:

```cpp
#include <gtest/gtest.h>

class IsPrimeTest : public ::testing::TestWithParam<int> {
protected:
    bool isPrime(int n) {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0 || n % 3 == 0) return false;
        for (int i = 5; i * i <= n; i += 6) {
            if (n % i == 0 || n % (i + 2) == 0)
                return false;
        }
        return true;
    }
};

TEST_P(IsPrimeTest, TestPrimes) {
    int num = GetParam();
    EXPECT_TRUE(isPrime(num));
}

INSTANTIATE_TEST_SUITE_P(
    PrimeNumbers,
    IsPrimeTest,
    ::testing::Values(2, 3, 5, 7, 11, 13, 17, 19, 23, 29)
);

TEST_P(IsPrimeTest, TestNonPrimes) {
    int num = GetParam();
    EXPECT_FALSE(isPrime(num));
}

INSTANTIATE_TEST_SUITE_P(
    NonPrimeNumbers,
    IsPrimeTest,
    ::testing::Values(1, 4, 6, 8, 9, 10, 12, 14, 15)
);
```

### Value Ranges

```cpp
INSTANTIATE_TEST_SUITE_P(
    RangeTest,
    MyTest,
    ::testing::Range(0, 10)  // 0, 1, 2, ..., 9
);

INSTANTIATE_TEST_SUITE_P(
    ValuesInTest,
    MyTest,
    ::testing::ValuesIn(my_vector)
);

INSTANTIATE_TEST_SUITE_P(
    CombineTest,
    MyTest,
    ::testing::Combine(
        ::testing::Values("a", "b"),
        ::testing::Values(1, 2, 3)
    )
);
```

### Type-Parameterized Tests

Test generic code with different types:

```cpp
template <typename T>
class QueueTest : public ::testing::Test {
protected:
    std::queue<T> q;
};

using MyTypes = ::testing::Types<int, double, std::string>;
TYPED_TEST_SUITE(QueueTest, MyTypes);

TYPED_TEST(QueueTest, PushAndPop) {
    TypeParam val = TypeParam();
    this->q.push(val);
    EXPECT_EQ(this->q.size(), 1);
    this->q.pop();
    EXPECT_TRUE(this->q.empty());
}
```

### Death Tests

Test code that should terminate:

```cpp
TEST(DeathTest, ExitsOnInvalidInput) {
    EXPECT_DEATH(
        {
            int* ptr = nullptr;
            *ptr = 42;  // Should crash
        },
        ".*"  // Regex for expected error message
    );
}

TEST(DeathTest, AssertsOnCondition) {
    ASSERT_DEATH(
        assert(false && "This should fail"),
        "This should fail"
    );
}
```

### Matchers (Google Mock)

```cpp
#include <gmock/gmock.h>

using ::testing::HasSubstr;
using ::testing::StartsWith;
using ::testing::EndsWith;
using ::testing::MatchesRegex;

TEST(StringTest, UsingMatchers) {
    std::string str = "Hello World";
    
    EXPECT_THAT(str, HasSubstr("World"));
    EXPECT_THAT(str, StartsWith("Hello"));
    EXPECT_THAT(str, EndsWith("World"));
    EXPECT_THAT(str, MatchesRegex("Hello.*"));
}
```

---

## Best Practices

### 1. Test Naming

```cpp
// Good: Descriptive names
TEST(StackTest, PushIncreasesSize)
TEST(StackTest, PopReturnsTopElement)
TEST(StackTest, PopOnEmptyStackThrows)

// Bad: Vague names
TEST(StackTest, Test1)
TEST(StackTest, TestPop)
```

### 2. One Assertion Per Concept

```cpp
// Good: Each test checks one behavior
TEST(VectorTest, PushBackIncreasesSize) {
    std::vector<int> v;
    v.push_back(1);
    EXPECT_EQ(v.size(), 1);
}

TEST(VectorTest, PushBackAddsElement) {
    std::vector<int> v;
    v.push_back(42);
    EXPECT_EQ(v[0], 42);
}

// Bad: Multiple unrelated assertions
TEST(VectorTest, TestPushBack) {
    std::vector<int> v;
    v.push_back(42);
    EXPECT_EQ(v.size(), 1);
    EXPECT_EQ(v[0], 42);
    EXPECT_EQ(v.capacity() >= 1, true);
}
```

### 3. Arrange-Act-Assert Pattern

```cpp
TEST(BankAccountTest, WithdrawalReducesBalance) {
    // Arrange
    BankAccount account;
    account.deposit(100.0);
    
    // Act
    account.withdraw(30.0);
    
    // Assert
    EXPECT_DOUBLE_EQ(account.getBalance(), 70.0);
}
```

### 4. Use Fixtures for Common Setup

```cpp
class FileSystemTest : public ::testing::Test {
protected:
    void SetUp() override {
        // Create temp directory
        fs::create_directory("test_tmp");
    }
    
    void TearDown() override {
        // Clean up
        fs::remove_all("test_tmp");
    }
};
```

### 5. Test Edge Cases

```cpp
TEST(DivisionTest, EdgeCases) {
    Calculator calc;
    
    // Zero
    EXPECT_DOUBLE_EQ(calc.divide(0, 5), 0.0);
    
    // Negative numbers
    EXPECT_DOUBLE_EQ(calc.divide(-10, 2), -5.0);
    EXPECT_DOUBLE_EQ(calc.divide(10, -2), -5.0);
    
    // Division by zero
    EXPECT_THROW(calc.divide(5, 0), std::invalid_argument);
}
```

---

## Real-World Examples

### Example 1: Testing a String Parser

```cpp
#include <gtest/gtest.h>
#include <string>
#include <vector>

class CSVParser {
public:
    std::vector<std::string> parse(const std::string& line, char delimiter = ',') {
        std::vector<std::string> tokens;
        std::string token;
        
        for (char c : line) {
            if (c == delimiter) {
                tokens.push_back(token);
                token.clear();
            } else {
                token += c;
            }
        }
        tokens.push_back(token);
        return tokens;
    }
};

class CSVParserTest : public ::testing::Test {
protected:
    CSVParser parser;
};

TEST_F(CSVParserTest, ParsesSimpleLine) {
    auto result = parser.parse("a,b,c");
    ASSERT_EQ(result.size(), 3);
    EXPECT_EQ(result[0], "a");
    EXPECT_EQ(result[1], "b");
    EXPECT_EQ(result[2], "c");
}

TEST_F(CSVParserTest, ParsesEmptyFields) {
    auto result = parser.parse("a,,c");
    ASSERT_EQ(result.size(), 3);
    EXPECT_EQ(result[0], "a");
    EXPECT_EQ(result[1], "");
    EXPECT_EQ(result[2], "c");
}

TEST_F(CSVParserTest, HandlesCustomDelimiter) {
    auto result = parser.parse("a|b|c", '|');
    ASSERT_EQ(result.size(), 3);
    EXPECT_EQ(result[0], "a");
}

TEST_F(CSVParserTest, HandlesEmptyString) {
    auto result = parser.parse("");
    ASSERT_EQ(result.size(), 1);
    EXPECT_EQ(result[0], "");
}
```

### Example 2: Testing a Cache

```cpp
#include <gtest/gtest.h>
#include <unordered_map>
#include <list>

template<typename K, typename V>
class LRUCache {
private:
    size_t capacity;
    std::list<std::pair<K, V>> items;
    std::unordered_map<K, typename std::list<std::pair<K, V>>::iterator> map;

public:
    LRUCache(size_t cap) : capacity(cap) {}
    
    V get(const K& key) {
        auto it = map.find(key);
        if (it == map.end()) {
            throw std::out_of_range("Key not found");
        }
        
        // Move to front
        items.splice(items.begin(), items, it->second);
        return it->second->second;
    }
    
    void put(const K& key, const V& value) {
        auto it = map.find(key);
        
        if (it != map.end()) {
            // Update existing
            it->second->second = value;
            items.splice(items.begin(), items, it->second);
            return;
        }
        
        // Add new
        if (items.size() >= capacity) {
            // Evict LRU
            auto last = items.back();
            map.erase(last.first);
            items.pop_back();
        }
        
        items.emplace_front(key, value);
        map[key] = items.begin();
    }
    
    size_t size() const { return items.size(); }
};

class LRUCacheTest : public ::testing::Test {
protected:
    LRUCache<int, std::string> cache{3};
};

TEST_F(LRUCacheTest, PutAndGet) {
    cache.put(1, "one");
    EXPECT_EQ(cache.get(1), "one");
}

TEST_F(LRUCacheTest, EvictsLRU) {
    cache.put(1, "one");
    cache.put(2, "two");
    cache.put(3, "three");
    cache.put(4, "four");  // Should evict 1
    
    EXPECT_THROW(cache.get(1), std::out_of_range);
    EXPECT_NO_THROW(cache.get(2));
    EXPECT_NO_THROW(cache.get(3));
    EXPECT_NO_THROW(cache.get(4));
}

TEST_F(LRUCacheTest, UpdatesExisting) {
    cache.put(1, "one");
    cache.put(1, "ONE");
    EXPECT_EQ(cache.get(1), "ONE");
    EXPECT_EQ(cache.size(), 1);
}

TEST_F(LRUCacheTest, AccessUpdatesLRU) {
    cache.put(1, "one");
    cache.put(2, "two");
    cache.put(3, "three");
    
    cache.get(1);  // Make 1 most recent
    cache.put(4, "four");  // Should evict 2, not 1
    
    EXPECT_NO_THROW(cache.get(1));
    EXPECT_THROW(cache.get(2), std::out_of_range);
}
```

### Example 3: Parameterized Test for Sorting Algorithms

```cpp
#include <gtest/gtest.h>
#include <vector>
#include <algorithm>
#include <functional>

class SortTest : public ::testing::TestWithParam<
    std::tuple<std::vector<int>, std::vector<int>>> {
};

void bubbleSort(std::vector<int>& arr) {
    for (size_t i = 0; i < arr.size(); i++) {
        for (size_t j = 0; j < arr.size() - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                std::swap(arr[j], arr[j + 1]);
            }
        }
    }
}

TEST_P(SortTest, BubbleSortWorks) {
    auto [input, expected] = GetParam();
    bubbleSort(input);
    EXPECT_EQ(input, expected);
}

INSTANTIATE_TEST_SUITE_P(
    SortingTests,
    SortTest,
    ::testing::Values(
        std::make_tuple(
            std::vector<int>{5, 2, 8, 1, 9},
            std::vector<int>{1, 2, 5, 8, 9}
        ),
        std::make_tuple(
            std::vector<int>{1, 2, 3},
            std::vector<int>{1, 2, 3}
        ),
        std::make_tuple(
            std::vector<int>{3, 2, 1},
            std::vector<int>{1, 2, 3}
        ),
        std::make_tuple(
            std::vector<int>{},
            std::vector<int>{}
        ),
        std::make_tuple(
            std::vector<int>{42},
            std::vector<int>{42}
        )
    )
);
```

---

## Running Tests

### Command Line Options

```bash
# Run all tests
./my_tests

# Run specific test suite
./my_tests --gtest_filter=CalculatorTest.*

# Run specific test
./my_tests --gtest_filter=CalculatorTest.Addition

# Run tests matching pattern
./my_tests --gtest_filter=*Addition*

# Exclude tests
./my_tests --gtest_filter=-CalculatorTest.Slow*

# Repeat tests
./my_tests --gtest_repeat=10

# Shuffle tests
./my_tests --gtest_shuffle

# List tests without running
./my_tests --gtest_list_tests

# Verbose output
./my_tests --gtest_print_time=1

# Stop on first failure
./my_tests --gtest_break_on_failure
```

### Output Formats

```bash
# XML output for CI
./my_tests --gtest_output=xml:test_results.xml

# JSON output
./my_tests --gtest_output=json:test_results.json
```

---

## Integration with CMake/CTest

**CMakeLists.txt**
```cmake
enable_testing()

add_executable(my_tests 
    test_calculator.cpp
    test_parser.cpp
    test_cache.cpp
)

target_link_libraries(my_tests 
    GTest::gtest_main
    my_library
)

gtest_discover_tests(my_tests)
```

Run with CTest:
```bash
ctest --output-on-failure
ctest -V  # Verbose
ctest -R Calculator  # Run tests matching regex
```

---

## Summary

Google Test is a powerful framework that makes C++ testing straightforward:

1. **Easy setup** with CMake FetchContent
2. **Rich assertions** for all data types
3. **Test fixtures** for code reuse
4. **Parameterized tests** for data-driven testing
5. **Death tests** for crash testing
6. **Excellent integration** with build systems

### Next Steps

- Practice with the examples above
- Read the [official documentation](https://google.github.io/googletest/)
- Explore Google Mock for mocking dependencies
- Integrate with your CI/CD pipeline
- Aim for high test coverage in your projects

Happy testing! 🧪