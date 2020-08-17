---
layout: post
title:  "Basic Setup for GTest and CMake (Platform Independent)"
date:   2019-03-23 23:00:00 +0000
categories: c++ cmake tutorial
tags: c++ cmake tutorial
---

This article is just a quick guide to get started on a `gtest` project. Since `gtest` is a compiled library, there is just way too much headaches with platform depdency or how things stuff is set up. Plus, if you use any CI system, there is more headache to get your CI system also set up. With this article, this will set it up easily and started with `gtest` and without having to muck around with the platform.

We discuss how to set up the basic `gtest` in a platform independent way with `cmake` in a simple, platform indepdendent way. We will use `git` and the `submodule` command to pull in google test. Windows does not have `gtest` in a specific place. A lot of Linux won't have the `gtest` package installed and build, or installed in a strange place. The goal is just to bypass all that headache and get started as soon as possible.

## GTest Hello World 

This is the basic google test template to get started with. This is what we would use for our `hellotest.cpp` file.

```cpp
#include <gtest/gtest.h>

TEST(TestTest, EmptyTest) {
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

## Git Setup

You have to turn the project into a `git` project first.

```bash
git init .
```

and then add the `gtest` as a submodule

```bash
git submodule add https://github.com/google/googletest
```

You should now see a copy of the google test directory inside your repo.

## CMake Setup

We can just use the `include_subdirectory` command in `cmake` to add the `gtest` into our project. There is just one small thing to take care of. Since the project was designed to also install gtest into the system and other things, we have to turn the `INSTALL_GTEST` option off. You can also look at the `googletest` `CMakelists.txt` file to look at all the options.

When we add `gtest` to our `target_link_libraries`, it gets all the required sub-libraries and include directories.

In the end, suppose our program is called `hellotest` and our `cmake` file would look like this

```cmake
cmake_minimum_required(VERSION 3.5)
project(hellotest)

set(CMAKE_CXX_STANDARD 14)

set(INSTALL_GTEST OFF)
add_subdirectory(googletest)

add_executable(hellotest test.cpp)
target_link_libraries(hellotest gtest)
```

## Google Benchmark

Using similar ideas, we can also add Google benchmark. 

### Git Submodule

For git, we have to make the google benchmark into a submodule.

```git
git submodule add https://github.com/google/benchmark
```

### CMake Changes

We first add all the relevant changes to add the benchmark into the project. The `BENCHMARK_ENABLE_TESTING` will disable requiring `GTest`. 

```cmake
set (BENCHMARK_ENABLE_INSTALL OFF)
set (BENCHMARK_ENABLE_TESTING OFF)
add_subdirectory(benchmark)
```

For the target itself, let us assume it is `hellbench.cpp` below and we add

```cmake
add_executable(hellobench hellobench.cpp)
target_link_libraries(hellobench benchmark)
```

### C++ Test Code

The basic test file that is from the Google benchmark read-me page:

```cpp
#include <benchmark/benchmark.h>

static void BM_StringCopy(benchmark::State& state) {
    std::string x = "hello";
    for (auto _ : state)
        std::string copy(x);
}
BENCHMARK(BM_StringCopy);

BENCHMARK_MAIN();
```

## Conclusion

This is the basic google test setup. It can run on Windows, Linux and on most CI servers without having to make any assumption about the system.

The project using these ideas as a skeleton project is available in this github repo (https://github.com/mochan-b/cmake-skeleton).
