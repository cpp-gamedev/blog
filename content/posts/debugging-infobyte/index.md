---
title: Debugging
date: 2020-04-24
description: C++ build configurations and debugging
categories: ['posts']
tags: ['infobytes', 'c++']
---

### Debugging Symbols

When invoking a C++ compiler, several options/flags can be passed, including optimisation level, pre-processor defines, and whether to include **debugging symbols** in the binary. 
These symbols are effectively mappings between binary addresses and source code (file, line number, symbol name, etc), which a compatible **debugger** can use. 
Including debugging symbols dramatically increases the size of the resulting binary, often by orders of magnitude.

A debugger (given an executable with debugging symbols) offers many useful features, such as stepping through code one line at a time, setting breakpoints, watching variables, changing values stored in memory, the entire call stack (when broken), etc. 
`g++` / `clang++` will embed debugging symbols when `-g` is passed as a compiler flag/option (`/DEBUG:FULL` for MSVC).
Most C++ runtimes also provide a "debugging mode", which enables bounds checking on containers, sets dangling pointers to a special value, etc., at the cost of performance. 
This is enabled on `libstdc++` by defining `_GLIBCXX_DEBUG`, either via a compiler option or directly in every source file (`/MTd` or `/MDd` option for MSVC).

### Build Configurations
A set of compiler options (and the source files) can be bundled together as a **build configuration** on most build tools, such that building that configuration will automatically pass the relevant options to the compiler when compiling a source file.
In CMake this is expressed via `CMAKE_BUILD_TYPE` (except for multi-configuration generators, like Visual Studio / Xcode); the defaults are:
- `Debug` (debugging symbols are added by default, but `_GLIBCXX_DEBUG` is _not_ defined by CMake!)
- `Release` (optimised for speed)
- `RelWithDebInfo` (with debug symbols)
- `MinSizeRel` (optimised for size)

Because of the overhead of symbols and debug mode, these are generally disabled by default, except for `Debug` configurations.

_Note: the most common debugger on Linux is `gdb`, while `clang` ships with `lldb` (both debuggers are cross-compatible)._

#### Resources
* [`gdb` in 60 seconds](https://youtu.be/mfmXcbiRs0E)

