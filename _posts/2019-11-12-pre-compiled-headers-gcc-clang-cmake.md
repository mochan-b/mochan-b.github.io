---
layout: post
title: "Using pre-compiled headers in GCC/Clang using CMake and usage in Catch2"
date: 2019-11-12
categories: c++
---

Pre-compiled headers can speed up the total compilation times. As `gcc` compiles each C++ file, the pre-processor adds each and every header file into a big file and then compiles that big file. In most cases, the same file is getting compiled multiple times over and over again. If we can process and compile the header files and use the results, we can save a considerable amount of time.

Project files are located [here](https://github.com/mochan-b/pch-cmake).

## Pre-compiled Headers in GCC

Suppose we have the follwing header file `hello.h` and source file `main.cpp`

```cpp
#include <iostream>

template<typename T>
void say(T t) {
	std::cout << t << std::endl;
}
```

```cpp
#include "hello.h"

int main() {
	say("hello world");
}
```

Obviously, using `g++ main.cpp -o main` will create the output file.

Now, let us ask `g++` with `-H` flag that outputs all the header files that is included using the command `g++ -H main.cpp -o main` and the result is a long list of header files.

```bash
. hello.h
.. /usr/include/c++/8/iostream
... /usr/include/x86_64-linux-gnu/c++/8/bits/c++config.h

```

How if we issue `g++ hello.h`, we see that it creates a file called `hello.h.gch`. And, then again running `g++ -H main.cpp -o main` results in the following output:

```bash
! hello.h.gch
 main.cpp
```
and that it is using the pre-compiled header.

Using the pre-compiled headers reduces the compilation time of the above example from _0.38s_ to _0.15s_. However, it does take _0.6s_ to build the pre-compiled header.

### Updating in the header files

If I change the header to add a tab before the `say` function, then the above command will only use the pre-compiled header that was previously generated and not give the right answer. If we regenerate the pre-compiled header, then it will generate the right answer.

```cpp
#include <iostream>

template<typename T>
void say(T t) {
	std::cout << '\t' << t << std::endl;
}
```

This is very dangerous situation since we have the pre-compiled header differing from the header and might lead to frustrations.

We will use CMake to fix the problem so that when `hello.h` is updated, then the pre-compiled header is also re-generated.

## Pre-Compiled Headers in Clang

Clang works in a similar way to g++. We first create the precompiled header file. The following command produces the `hello.h.pch` file as specified.

```bash
clang++ -stdlib=libstdc++ -x c++-header hello.h -o hello.h.pch
```

Different from `g++`, we have to remove the `#include "hello.h"` from the source file `main.cpp` and add the include in the command line. So, our `main.cpp` will look like only.

```cpp
int main() {
        say("hello world");
}
```

We build the binary using the following command line. Note the inclusion of the header file using the command line.

```bash
clang++ -stdlib=libstdc++ -include hello.h main.cpp -o main
```

Unlike `gcc` where the usage of the pre-compiled header is almost invisible, we have to explicitly remove it from the source code and add it to the command line. 

Another thing to note is that if I modify the header and use an outdated pre-compiled header file, I get an error from clang. Whereas g++ would happily produce an outdated binary, clang will throw an error and stop the compilation.

```bash
fatal error: file 'hello.h' has been modified since the precompiled header 'hello.h.pch' was built
note: please rebuild precompiled header 'hello.h.pch'
1 error generated.
```

### Compilation Times

Similarly as g++, clang also reduced the compilation times after using pre-compiled header. For me, to build the pre-compiled header took 0.353s. To build main with the pre-compiled header took m0.179s. Without pre-compiled headers, clang took 0.381s. 

## CMake Pre-Compiled Headers

CMake uses the `clang` method (which would also work for g++). We use the `target_precompile_headers` in CMake to use pre-compiled headers. The include in the `main.cpp` of `hello.h` has to be removed.

Thus, our CMake file for our project would be 

```cmake
cmake_minimum_required(VERSION 3.16)
project(pch-cmake)

add_executable(main main.cpp)
target_precompile_headers(main PRIVATE hello.h)
```

When we build the project, we get the following output showing that the pre-compiled header is built.

``` bash
[ 33%] Building CXX object CMakeFiles/main.dir/cmake_pch.hxx.gch
[ 66%] Building CXX object CMakeFiles/main.dir/main.cpp.o
[100%] Linking CXX executable main
[100%] Built target main
```

Building by clang also works and get the same output. We can use the `CC` and `CXX` variables to specify the compiler.

```bash
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++
cmake ..
-- The C compiler identification is Clang 8.0.0
-- The CXX compiler identification is Clang 8.0.0
-- Check for working C compiler: /usr/bin/clang
-- Check for working C compiler: /usr/bin/clang - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/clang++
-- Check for working CXX compiler: /usr/bin/clang++ - works

...
```

Using `clang` produces a `pch` file rather than a `gch` file.

```bash
[ 33%] Building CXX object CMakeFiles/main.dir/cmake_pch.hxx.pch
[ 66%] Building CXX object CMakeFiles/main.dir/main.cpp.o
[100%] Linking CXX executable main
[100%] Built target main
```

## Pre-Compiled Catch2 Header for Faster Unit Testing

Lets try to use pre-compiled headers on `catch2` unit testing framework that is header only and has a long compile time.

Our project has `Catch2` as a submodule from the `github` repo. Thus, the single include header file is at the location `Catch2/single_include/catch2/catch.hpp`.

The very basic `Catch2` test is the following:
```cpp
#define CATCH_CONFIG_MAIN  
#define CATCH_CONFIG_FAST_COMPILE
#define CATCH_CONFIG_DISABLE_MATCHERS
#include "catch.hpp"

TEST_CASE( "Two and Two is Four", "[2+2=4]" ) {
    REQUIRE( 2+2 == 5 );
}
```

The corresponding `cmake` entry is the standard header-only usage.
```cmake
add_executable(catch-test2 catch-test2.cpp)
target_include_directories(catch-test2 PUBLIC Catch2/single_include/catch2/)
```

Compilation and re-compilation all takes about 5.6s

To use the pre-compiled headers, we remove all the header information from the main file and we are left with the following.

```cpp
TEST_CASE( "Two and Two is Four", "[2+2=4]" ) {
    REQUIRE( 2+2 == 3 );
}
```

and the corresponding CMake to compile the project

```cmake
add_executable(catch-test catch-test.cpp)
target_compile_definitions(catch-test PRIVATE CATCH_CONFIG_MAIN CATCH_CONFIG_FAST_COMPILE CATCH_CONFIG_DISABLE_MATCHERS)
target_precompile_headers(catch-test PRIVATE Catch2/single_include/catch2/catch.hpp)
```

The first compile to generate the pre-compiled headers take 6.3s and the subsequent recompilations with changes in the test take 4.7s. Compared to not using pre-compiled headers, we save about 0.9-1 seconds each time we modify the test and re-run the test.

## Conclusion

Using pre-compiled headers reduces the compilation time. With the support by `cmake` for pre-compiled headers, there is an easy way to integrate pre-compiled headers into our projects.

## References

- [`g++` pre-processor options/flags](https://gcc.gnu.org/onlinedocs/gcc-3.1.1/gcc/Preprocessor-Options.html#Preprocessor%20Options)
- [`gcc` documentation of using pre-compiled headers](https://gcc.gnu.org/onlinedocs/gcc/Precompiled-Headers.html)
- [`clang` documentation of pre-compiled headers](https://clang.llvm.org/docs/UsersManual.html#usersmanual-precompiled-headers)
- [`cmake` reference on `target_precompile_headers`](https://cmake.org/cmake/help/latest/command/target_precompile_headers.html)
- [`catch2` github page](https://github.com/catchorg/Catch2)
- [Project files for this post](https://github.com/mochan-b/pch-cmake)
