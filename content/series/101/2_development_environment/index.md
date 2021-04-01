---
title: Series 101-2 Development Environment
date: 2021-03-30
description: Setting up a C++ developing and debugging environment
categories: ["series"]
tags: ["101", "c++"]
---

C++ compilers take various flags and options in addition to the source files (and output), to control a multitude of options. The most relevant one here is one which controls whether the compiler embeds **debugging symbols** into / alongside the binary it builds - these are mappings between generated code and source code/line numbers, which can be used by debuggers to associate both, eg for setting breakpoints. Most build systems (Make, Ninja, MSBuild, Xcode, etc) group a collection of these options and flags into a **build configuration**, and most multi-generator build systems (MSBuild, Xcode) provide a handful of in-built defaults (Debug, Release). In CMake, this is controlled by `CMAKE_BUILD_TYPE`, which when explicitly set to `Debug` (as in the previous post) instructs CMake to configure that particular build directory as if it were using a Debug build configuration, enabling debugging symbols on all generators. Debuggers (GDB, LLDB, MSVC, etc) can use these symbols to navigate and manipulate the binary at runtime. The basics of debugging are platform and OS dependent, and generally out of scope of this post, which primarily demonstrates hooking up a native debugger to a text editor like VSCode and using it via GUI.

References:

- [GDB in 60 seconds](https://www.youtube.com/watch?v=mfmXcbiRs0E)

> _[**Previous: Getting Started**](https://cpp-gamedev.netlify.app/series/101/1_getting_started/)_

### Targets

1. Setup clangd for "Intellisense" (if supported by your editor)
1. Amp up compiler warnings (through CMake)
1. Setup a text editor and scripts to build / debug

### Prerequisites

[**Visual Studio Code**](https://code.visualstudio.com/) is highly recommended and assumed as the editor unless stated otherwise throughout the rest of this series; although if you are comfortably fluent with editors and shells, feel free to adapt this guide to your preferred ones. Similarly, `ninja` and `clang++` are assumed as the build tool and compiler - these work on both Windows and Linux (and Android, technically), and require mostly identical workflows and setups. However, Visual C++ (MSVC) is the defacto compiler and MSBuild the build system, on Windows - this is not trivial to integrate into a CMake workflow, though not impossible either, but beyond the scope of this guide. `clang++` provides a `g++` like front-end but uses the native standard library runtime on both platforms, meaning it will link to libc++ / libstdc++ / MSVC, and those are required for it to work correctly (aka, on Windows, you need to install Visual Studio and its C++ toolset regardless of which compiler you pick).

**VSCode C++ Guides**

- [Linux](https://code.visualstudio.com/docs/cpp/config-linux)
- [Windows](https://code.visualstudio.com/docs/cpp/config-msvc)
- [Debugging](https://code.visualstudio.com/docs/cpp/cpp-debug)

> _**Note:** While LLDB (the debugger that ships with LLVM) can be made to work with VSCode on both OSs, my experience has been much better with `gdb` and `cppdbg`, so this guide will use that approach._

**Extensions**

- [Microsoft C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) : Make sure to disable C/C++ Intellisense and squiggly lines etc - clangd will take over this part
- [clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)

#### clangd

[**Clangd**](https://clangd.llvm.org/) is a language server that integrates fairly well with VSCode (and many other editors/plugins), facilitating "Intellisense" while coding, with features such as code completion, error highlights and tooltips, code navigation and contextual renaming, etc. To most effectively use the clangd language server and VSCode's features, a project should provide a `compile_commands.json`, generated for the current build configuration (`Debug`, except in rare circumstances) in its root directory. CMake is capable of generating this for any configuration it produces, which is done by setting `CMAKE_EXPORT_COMPILE_COMMANDS=1` in `CMakeCache.txt` (or passing `-DCMAKE_EXPORT_COMPILE_COMMANDS=1` when configuring for the first time), and configuring. Symlinking this JSON to the project root will work (on Linux); however, a slightly better approach is to have CMake itself copy the JSON if it exists, after configuration / during generation. This will also trigger VSCode to prompt you to reload clangd when it detects the file has been modified (which doesn't always work with symlinks).

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

Check clangd's logs in VSCode's **Output** window to confirm that it isn't reporting any errors.

> _**Note:** CMake, `compile_commands.json`, and clangd don't work well together when using **precompiled headers**; in such cases create a dummy config as a copy of `Debug` but with PCH off, and copy and use the JSON generated there._

#### clang-format

Most C++ programmers spend a considerable amount of time meticulously formatting their precious code, until they make an acquaintance with an autoformatter - of which `clang-format` is the indisputed champion - after which they pretty much never hand-format code again. This is the secret behind the reams of impossibly perfectly formatted and documented library code, for example throughout the standard library headers.

It works by way of a configuration file in the root, much like `compile_commands.json`, called `.clang-format`, which the `clang-format` binary reads for the configured [**format style options**](https://clang.llvm.org/docs/ClangFormatStyleOptions.html). To start using it, create a `.clang-format` in the project root and select "Format document with clangd" through VSCode's command palette. VSCode also offers an option to auto-format documents on save (using the configured formatter).

#### Compiler Warnings and clang-tidy

Since C++ standards are generally very meticulous about backwards compatibility, not-so-great language and library design decisions made in the past cannot easily be rectified through breaking changes, and older programs must not suddenly stop compiling with a new compiler (provided they weren't engaging in unspecified behaviour in the first place). Ergo, there are a few language potholes and traps where the compiler can only warn you - cannot issue an error and stop (unless you configure it to be that way for a specific set of / all warnings), and which is why modern C++ development is greatly aided by turning up compiler warnings and using linters that perform static analysis outside compilation (perhaps even risking some false positive warnings). `clang++` offers a `g++`-like front-end, and accepts all those compiler flags (and many more); at the bare minimum it is highly recommended to set at least the following, for **every single C++ project you author**:

```cmake
target_compile_options(foo PUBLIC -Wall -Wextra)
```

`clang-tidy` can be invoked through the command line, though it is also available as a third-party [editor plugin](https://marketplace.visualstudio.com/items?itemName=notskm.clang-tidy) for VSCode (not authored by LLVM). There should be more integrated support for it within a few years.

> _**Recommended:** [CppCon 2017: Kate Gregory "10 Core Guidelines You Need To Start Using Now"](https://www.youtube.com/watch?v=XkDEzfpdcSg)_

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

Unlike building, which just involved having the editor invoke a customised shell command, GUI debugging requires deeper integration with the editor, to orchestrate setting / removing breakpoints, watching variables / modifying memory, etc. VSCode facilitates this via the C/C++ plugin, which provides the front-end capabilities on the editor, and hooks up to a configured native debugger as the backend. This is controlled through `./.vscode/launch.json`, which drives the **Run and Debug** tab. To create the file, switch to the tab and select "create a launch.json file", and pick "C/C++" as the template ("GDB/LLDB" for Linux, "Windows" for Windows). Set `name` to be the label in the **Run and Debug** tab; assuming the binary to be at `./out/debug/foo`, modify `program` to be `${workspaceFolder}/out/debug/foo` (and any other relevant fields). Click the play button / press F5 to start debugging. Refer to the [**launch.json reference**](https://code.visualstudio.com/docs/cpp/launch-json-reference) for more details.

> _**Note:** Make sure `CMAKE_BUILD_TYPE` is set to `Debug` in `out/debug/CMakeCache.txt`, else there will be no debugging symbols generated during a build!_

Linux:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "(debug) Launch",
      "type": "cppdbg",
      "request": "launch",
      "program": "${workspaceFolder}/out/debug/foo",
      "cwd": "${workspaceFolder}",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ]
    }
  ]
}
```

Windows:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "(debug) Launch",
      "type": "cppvsdbg",
      "request": "launch",
      "program": "${workspaceFolder}/out/debug/foo.exe",
      "cwd": "${workspaceFolder}"
    }
  ]
}
```
