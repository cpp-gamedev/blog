---
title: Topics Outline
date: 2020-02-26
description: Outline of common-ground topics for project collaborators to cover
categories: ['collab']
tags: ['projects']
---

### Pre-3D Topics

**Environment Setup**
- CMake basics
- Dealing with paths - preprocessor vs working directory vs runtime directory, relative vs absolute
- Limiting symbols in global scope (everything is global by default)
- Debugging basics: embedding symbols, using breakpoints, watching variables, changing memory, etc
- Enabling useful compiler warnings

**I/O, OS, code scope**
- Generic types vs fixed-size types
- Manual memory management
- IO and type conversions: `<iostream>`, `std::getline`, `std::string`/`std::string_view`, `std::atoi`, etc
- Containers:  `std::stringstream`, `std::vector`, `std::unordered_map`, `std::unordered_set`, `std::list`, `std::deque`, `std::bitset`
- Files: `<fstream>`, `std::filesystem`
- Random generation: `<random>`
- Function templates

**Advanced C++**
- Move semantics
- Customization points (operator overloading, custom iterators, etc)
- `<algorithm>` (`find`, `find_if`, `for_each`, `any_of`, `rotate`, `transform`, `iota`)
- Function objects (functors) and lambdas (immediate vs deferred, lifetime issues with captures)
- `<functional>`, basic callback design
- `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`
- Class templates
- Explicit specializations, SFINAE
- Static (compile time) polymorphism
- Performance optimizations

**Game Fundamentals**
- Measuring time
- Input state machines
- Lifetime-aware callbacks ("Delegates")
- Numerical integration via `deltaTime`
- Linear interpolation
- Event loops / game loops
- Update method
- SFML, graphics best practices

**3D**

- Vectors: geometric interpretation, algebraic operations, multiplications
- Matrices: geometric interpretation, algebraic operations, multiplications, transformations, translations, rotations, scaling, etc
- Quaternions: geometric interpretation, algebraic operations, multiplications, interpolations, conversions to angles/matrices
- Cominbing matrix transformations
- Model View Matrix pipeline
- Simple problems in 3D space
- Native graphics APIs (OpenGL, DirectX, Vulkan)
- Depth buffers
- Shaders
