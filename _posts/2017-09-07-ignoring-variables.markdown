---
layout: post
title:  "Ignoring variables"
date:   2017-09-07
author: Emil Ohlsson
categories: c macros
---
One thing that I often miss from C++ is the ability to easily ignore parameters.
This is handy when you have an interface to adhere to, but you are not
interested in all the parameters.

In C++ you can do this by
```c++
int foo(int, int bar)
{
    return bar;
}
```
This will not generate any warning that the first parameter is unused.

C macros to the rescue!
```c
#define _CONCAT(x, y) x ## y
#define CONCAT(x, y) _CONCAT(x, y)
#define IGNORED(type) __attribute__((unused)) type CONCAT(ignored, __COUNTER__)
```
The `__COUNTER__` macro resolves an incremented number for each usage, which
allows the `IGNORED` macro to be used several times per line.

So, the example C++ example above can be implemented using
```c
int foo(IGNORED(int), int bar)
{
    return bar;
}
```
