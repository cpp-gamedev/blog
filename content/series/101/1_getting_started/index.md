---
title: Series 101-1 Getting Started
date: 2021-03-23
description: Authoring your first C++ project
categories: ["series"]
tags: ["101", "c++"]
---

C++ is a statically typed application/library programming language, where the source code is compiled (and linked) into a native binary for a particular target platform, like x64, ARMv8, etc (and usually also operating system, like Windows, GNU/Linux, Android, etc). The C++ standard only specifies the rules of the language and its behaviour on an **abstract machine**, and the **standard library**'s interface; compilers like Microsoft VC++, LLVM Clang, GCC, Apple C++ (and others) implement the standard and translate the source to actual platform instructions, and also provide the corresponding implementation of the standard library. Since there is no "standard compiler" or any other such tools, this guide picks those that have the widest coverage among the most popular platforms and OSs.

Please don't despair if none of this makes sense (yet), it will in due time. This series will strive to provide links and references to more detailed literature for topics which can prove to be complex rabbit holes. The posts in this series are designed to ramp up in complexity, depth, and nuance, as they progress; the first one having only the necessary and sufficient instructions to author, build, and run the most simple C++ project. Future posts will delve into debugging, using directories and multiple source files/headers, building and linking to (static) libraries, interfacing with source control, basic unit testing, concepts of continuous integration, etc.

> _**Note:** This series follows C++17_

### Requirements

1. Platform/OS: x64 Windows (10+), x64/ARM64 GNU/Linux (5.x+, any distribution): while other platforms / older versions may work, this guide has only been verified on these
   1. (Windows only): Visual Studio 2019 and its [**Desktop Development with C++** kit](https://docs.microsoft.com/en-us/cpp/build/media/vscpp-concierge-choose-workload.gif?view=msvc-160)
1. **[CMake 3.15+](https://cmake.org/download)** (Resources: [Reference](https://cmake.org/cmake/help/v3.20/) [Wikipedia](https://en.wikipedia.org/wiki/CMake), [Modern CMake](https://cliutils.gitlab.io/modern-cmake/modern-cmake.pdf))
1. **[Ninja 1.9+](https://github.com/ninja-build/ninja/releases)** (Resources: [Manual](https://ninja-build.org/manual.html), [Wikipedia](<https://en.wikipedia.org/wiki/Ninja_(build_system)>))
1. **[LLVM/Clang 9+](https://releases.llvm.org/download.html)** (Resources: [Manual](https://clang.llvm.org/docs/UsersManual.html), [Wikipedia](https://en.wikipedia.org/wiki/Clang))
1. **[clangd server](https://clangd.llvm.org/installation.html)** (Resources: [Configuration](https://clangd.llvm.org/config.html), [Wikipedia (LSP)](https://en.wikipedia.org/wiki/Language_Server_Protocol))
1. **Text Editor**: any, though VSCode is recommended and used in the rest of this series

> _Author's note: Why no MacOSX? Because I have no way to test/troubleshoot it... In general you'd follow the same process, just skip the generator/compiler selection and use the defaults (Xcode and Apple Clang)._

### Authoring an Executable

It helps to divide one's mindset into two "stages": that of authoring source (and scripts), and that of building/running/debugging it. There are some concerns that are only relevant in one stage, and others that matter more in one than the other. In practice, you will find yourself iterating and context switching between the two stages often, especially when debugging a problem; it is strongly recommended to respect the separation of concerns even during such frustrating times.

#### Project Workspace

Create a new directory for the project, somewhere, say `hello`

- This is the **project root** (`./`)
- The **author** of the project (you) must ensure to keep it agnostic of anything before/above `./` (where `hello` is located)
- This ensures that a **user** of the project (also you) can clone/download and build it anywhere they like
- Similarly, ideally the project should not require any environment variables etc to have been set up prior
- This is an example of an authoring concern that doesn't exist when building

#### Project Files

Create a new source file: `main.cpp`, and write the shortest possible C++ program:

```cpp
int main() {}
```

- This **_does not imply_** that `return thingy;` is optional in C++! It is **only** defined behaviour to skip it for `int main()` (implicitly returns `0`)
- Although this program seems to literally do nothing, it _does_ offer one affordance: it _returns `0`_ when run successfully (if you are acquainted with the shell, you may already realise how to exploit this)

Create a new file: `./CMakeLists.txt`

- This is a **CMake (project) script**
- Each project must have such a script at its root
- A project may contain other projects as subdirectories (advanced usage, not part of this post)

Add the following lines to the script, in order:

```cmake
cmake_minimum_required(VERSION 3.3)
```

- This establishes a minimum CMake version requirement for the project

```cmake
project(hello)
```

- This declares a new CMake project, it is just a name, but is conventionally also used as the name of the executable

```cmake
add_executable(${PROJECT_NAME} main.cpp)
```

- This instructs CMake to construct an executable **target** called `hello` (`${PROJECT_NAME}`, set above) using one source file: `main.cpp`

```cmake
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)
```

- This enables the C++17 standard for all source files in this project
- `PRIVATE` properties of a target do not propagate to others that link to it (nothing links to this target, so here it's just a matter of satisfying CMake's syntax)

At this point, your role as the **author** is complete.

### Configuring a Build

CMake is a meta build system (or build system build system), whose primary purpose is to be passed the "source directory" (location of root `CMakeLists.txt`) and other options, and to **generate** a build system at the provided "output directory". It supports various generators, of which this guide shall utilise Ninja. The simplest way to invoke CMake to do the above is to run this in the project root:

```
cmake .
```

If successful, CMake should log the following towards the end of the output:

```
Configuring done
Generating done
```

This will use the default generator and options for the detected platform / OS. It will also output all the generated files to the project root - this is known as an "in-source build", and is generally not recommended (as it pollutes the project root with generated files). The most important generated artefact is `CMakeCache.txt` - this is populated the first time a CMake generator is invoked for an output / build directory, and reused on subsequent runs. Sometimes a CMake configuration gets corrupted, often due to an attempt to reconfigure using a different generator / compiler / etc, which can be fixed by simply deleting the cache (`CMakeCache.txt`), and regenerating CMake for that output directory. By using a custom output subdirectory, the entire subdirectory can be deleted safely (in order to purge everything - cache, binaries, object files, etc) without affecting the project source files.

The way to specify a custom output / build directory is via the `-B` flag:

```
cmake . -B out
```

Depending on your platform / OS this may end up using one of the Makefiles or Visual Studio generators. Since you want to use Ninja, specify that via `-G Ninja`. Remember to purge the cache first:

```
cmake . -B out -G Ninja
```

One more aspect to customise is compiler selection; unfortunately there are no quick flags for this, and the options must be specified by passing the full CMake cache flag and its desired value. This is done via the following syntax: `-D<variable_name>=<desired_value>` (`D` stands for "define", like `#define`), and the two variables we want to set are:

```
CMAKE_C_COMPILER=clang
CMAKE_CXX_COMPILER=clang++
```

Ensure `clang --version` works as expected, else add the LLVM installation directory (`bin`) to your environment's `PATH`, and specify the final configuration:

```
cmake . -B out -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
```

As mentioned before, this is only required when configuring / generating an output directory for the first time (`CMakeCache.txt` does not exist); subsequent runs can be updated simply via `cmake out`.

> _Note: Technically the **build configuration** here is undefined (empty/blank); if you would like to enforce Debug, pass it via `-DCMAKE_BUILD_TYPE=Debug`, or add/modify the variable in `CMakeCache.txt` directly (and reconfigure). Build configurations and debugging will be explored in detail in a future post._

### Building and Running

Building a configured CMake project is quite simple:

```
cmake --build out
```

_Where_ the build system will output the binaries varies per generator; Ninja builds them following the source directory structure of the project, so in this case directly in `out`. The name of the executable will be what was specified as the target in `CMakeLists.txt` (`${PROJECT_NAME}` which is `hello`). Run it as you would any executable on your shell: `./out/hello`, `./out/hello.exe`, etc. Any program that returns `0` is considered to have "successfully completed" by all major shell environments, so you can confirm it works via: `./out/hello && echo SUCCESS!` or `./out/hello || echo FAILED!`.

### Subsequent Runs

Modify `main.cpp` to be:

```cpp
#include <iostream>

int main() {
  std::cout << "Hello world!\n";
}
```

Build the project:

```
cmake --build out
```

Run the updated binary.
