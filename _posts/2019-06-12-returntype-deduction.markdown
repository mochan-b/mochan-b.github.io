---
layout: post
title:  "Deduce Return Type of a Function"
date:   2019-06-12 23:00:00 +0000
categories: c++
tags: c++
---

C++ functions can have return type `auto` and it is not always obvious what the return type of a function is. The return type can be dependent on template parameters.

## Return type using decltype

Suppose we have the function `foo()` as follows:

```cpp
int foo()
{
  return 123;
}
```

To get the return type of `foo`, we simply use the following:

```cpp
using t = decltype(foo());
```

`t` now contains the return type.

We can use `t` as a type declaration as follows,

```cpp
t i = foo();
```

Note, that `decltype` is all done at compile time and does not call the method `decltype`.

## Getting the compiler to tell us type

The C++ compiler knows the type of each and every variable during compile time. However, it is not trivial to get the type of a variable from the compiler.

One of the tricks to get it to tell us the type involves producing a compiler error so that outputs the type in the compiler error.

```cpp
template<typename T>
class error;
```

and using the above to produce the error during compilation.

```cpp
using t = decltype(foo());
error<t>();
```

The output of compilation is following error that tells the type of t.

```cpp
error: invalid use of incomplete type ‘class error<int>’
```

We thus know that the type of `t` is `int`.

## Printing out the type

The first instinct is to print out the type and C++ does provide the function `typeid` and the `name()` function to convert it to string. However, it strips out things like `const` and references `&`.

The other trick is to use the `__PRETTY_FUNCTION__` which prints out the contents of pretty version of a function at compile time. This also prints out the template type.

For example, using the function

```cpp
template <class T>
auto type_name()
{
  return __PRETTY_FUNCTION__;
}
```

and calling the function in this manner,

```cpp
using t = decltype(foo());
std::cout << type_name<t>() << std::endl;
```

the output in my system with `gcc` with `ubuntu` produces the following output:

```cpp
auto type_name() [with T = int]
```
and so, we know the type of `t` is `int`. 

## Functions with Paramters

Suppose we change our function `foo` in the following way, 

```cpp
int foo(int i)
{
  return 123+i;
}
```

and our method of using just `decltype(foo())` does not work since `foo` now has parameters. We can use `decltype(foo(2))`. However, if we don't really want to specify that we are going to use 2, we can use `std::declval` to not have to use a parameter.

```cpp
using t = decltype(foo(std::declval<int>()));
```

If we had two parameters for `foo` and looks like the following function, 

```cpp
int foo(int i, double j)
{
  return 123+i+j;
}
```

we can use the following:

```cpp
using t = decltype(foo(std::declval<int>(), std::declval<int>()));
```

### Templates

If we template `foo`, it just involves using `decltype` with how we would call `foo`.

```cpp
template<typename T>
T foo(int i, double j)
{
  return 123+i+j;
}
```

The way to call the `decltype` is as follows.

```cpp
using t = decltype(foo<double>(std::declval<int>(), std::declval<int>()));
```

### Conclusion

Getting the return type of a function just involves using `decltype` and `declval`. It is also useful to use some tricks to get the compiler to tell us what the actual types are before we use them.
