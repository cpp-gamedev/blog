---
title: Series 101-2 Development Environment
date: 2021-03-30
description: Setting up a C++ developing and debugging environment
categories: ["series"]
tags: ["101", "c++"]
---

C++ compilers take various flags and options in addition to the source files (and output), to control a multitude of options. The most relevant one here is one which controls whether the compiler embeds **debugging symbols** into / alongside the binary it builds - these are mappings between generated code and source code/line numbers, which can be used by debuggers to associate both, eg for setting breakpoints. Most build systems (Make, Ninja, MSBuild, Xcode, etc) group a collection of these options and flags into a **build configuration**, and most multi-generator build systems (MSBuild, Xcode) provide a handful of in-built defaults (Debug, Release). In CMake, this is controlled by `CMAKE_BUILD_TYPE`, which we explicitly set to `Debug` in the previous post - that instructed CMake to configure that particular build directory as if it were using a Debug build configuration, which enables debugging symbols on all generators. Debuggers (GDB, LLDB, MSVC, etc) can use these symbols to navigate and manipulate the binary at runtime. The basics of debugging are platform and OS dependent, and generally out of scope of this post, which primarily demonstrates hooking up a native debugger to a text editor like VSCode and using it via GUI.

References:

- [GDB in 60 seconds](https://www.youtube.com/watch?v=mfmXcbiRs0E)

> _[**Previous: Getting Started**](https://cpp-gamedev.netlify.app/series/101/1_getting_started/)_

### Targets

1. Setup clangd for "Intellisense" (if supported by your editor)
1. Amp up compiler warnings (through CMake)
1. Setup a text editor and scripts to build / debug

### Prerequisites

VSCode is highly recommended and assumed as the editor unless stated otherwise throughout the rest of this series; although if you are comfortably fluent with editors and shells, feel free to adapt this guide to your preferred ones. Similarly, `ninja` and `clang++` are assumed as the build tool and compiler - these work on both Windows and Linux (and Android, technically), and require pretty much identical workflows and setups. However, Visual C++ (MSVC) is the defacto compiler and MSBuild the build system, on Windows - this is not trivial to integrate into a CMake workflow, though not impossible either, but beyond the scope of this guide. `clang++` provides a `g++` like front-end but uses the native standard library runtime on both platforms, meaning it will link to libc++ / libstdc++ / MSVC, and those are required for it to work correctly (aka, on Windows, you need to install Visual Studio and its C++ toolset regardless of which compiler you pick).

VSCode Guides:

- [Linux](https://code.visualstudio.com/docs/cpp/config-linux)
- [Windows](https://code.visualstudio.com/docs/cpp/config-msvc)
- [Debugging](https://code.visualstudio.com/docs/cpp/cpp-debug)

> _**Note:** While LLDB (the debugger that ships with LLVM) can be made to work with VSCode on both OSs, my experience has been much better with `gdb` and `cppdbg`, so this guide will use that approach._

**Extensions**

- [Microsoft C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) : Make sure to disable C/C++ Intellisense and squiggly lines etc - clangd will take over this part
- [clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)

#### clangd

To most effectively use the clangd language server and VSCode's features, a project should provide a `compile_commands.json`, generated for the current build configuration (pretty much `Debug`, except in rare circumstances) in its root directory. CMake is capable of generating this for any configuration it produces, simply by setting `CMAKE_EXPORT_COMPILE_COMMANDS=1` in `CMakeCache.txt` (or passing `-DCMAKE_EXPORT_COMPILE_COMMANDS=1` when configuring for the first time), and configuring. Symlinking this JSON to the project root will work (on Linux); however, a slightly better approach is to have CMake itself copy the JSON if it exists, after configuration / during generation. This will also trigger VSCode to prompt you to reload clangd when it detects a change to the file (which is hit or miss with symlinks).

```cmake
# set the json path to a variable, it will be needed multiple times
set(CCJSON "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json")

# check if the file exists
if(EXISTS "${CCJSON}")
    # log a status message to confirm it works
	message(STATUS "Copying ${CCJSON} to project root")

    # copy the file
	file(COPY "${CCJSON}" DESTINATION "${PROJECT_SOURCE_DIR}")
endif()
```

Check clangd's logs in VSCode's Output window to confirm that it isn't reporting any errors.

> _**Note:** CMake, `compile_commands.json`, and clangd don't work well together when using **precompiled headers**; in such cases create a dummy config as a copy of `Debug` but with PCH off, and copy and use the JSON generated there._

#### clang-format

Most C++ programmers spend a considerable amount of time meticulously formatting their precious code, until they make an acquaintance with an autoformatter - of which `clang-format` is the indisputed champion - after which they pretty much never hand-format code again. This is the secret behind the reams of impossibly perfectly formatted and documented library code (if you have ever stumbled across `vector` by mistake, for example) you may have encountered. Well, not the documentation, that you still have to write by hand; but `clang-format` will format it appropriately, as per your configuration. To start using it, simply create a `.clang-format` in the project root and select "Format document with clangd" through VSCode's command palette. There is also an option to auto-format documents on save (ones for which formatters are available).

#### Compiler Warnings and clang-tidy

Since C++ is obsessed with backwards compatibility, not-great design decisions made in the past cannot easily be rectified through breaking language changes, and older programs must not suddenly stop compiling with a new compiler (provided they weren't engaging in unspecified behaviour in the first place). Ergo, there are a few potholes and traps where the compiler can only warn you - cannot issue an error and stop (unless you configure it to be that way for a specific set of / all warnings), and which is why modern C++ development is greatly aided by turning up compiler warnings and using linters that perform static analysis outside compilation (perhaps even risking some false positive warnings). Since `clang++` "pretends" to have a `g++` like front-end, it accepts all those compiler flags (and many more); at the bare minimum it is highly recommended to set at least the following, for **every single C++ project you author**:

```cmake
target_compile_options(foo PUBLIC -Wall -Wextra)
```

`clang-tidy` can be invoked through the command line, though it is also available as a third-party [editor plugin](https://marketplace.visualstudio.com/items?itemName=notskm.clang-tidy) for VSCode (not authored by LLVM). I expect there to be more integrated support for it within a few years.

### Building and Debugging

The core idea behind debugging is to spin off a command line task: `cmake --build <output_dir>` which can be invoked by some keyboard shortcut via the text editor. Debugging is a bit more involved, though command line debugging is simpler on Linux than Windows; setting up VSCode to use the native debugger (`gdb` or `cppdbg`) is relatively straightforward on both. On VSCode both these aspects are driven by a JSON file (in `./.vscode`) each.

#### Building

Assuming the output directory to be `out/debug` (to accommodate multiple build configurations in the same parent output directory), create a `tasks.json` in `./.vscode` and paste the following:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "build (debug)",
      "type": "shell",
      "command": "cmake",
      "args": ["--build", "out/debug"],
      "group": "build"
    }
  ]
}
```

This is valid for both platforms, all shells, all generators (including GNU Makefiles, Microsoft Visual Studio, etc): `cmake` handles all the platform specific details! Visit [this page]() for more information on the JSON file and format.

#### Debugging

WIP
