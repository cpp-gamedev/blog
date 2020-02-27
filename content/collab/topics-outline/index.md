---
title: Topics Outline
date: 2020-02-26
description: Outline of common-ground topics for project collaborators to cover
categories: ['collab']
tags: ['projects']
---

### C++ Basics
**Environment Setup**
<style>
  li { line-height: .2;}
</style>
 - CMake basics
 - Dealing with paths  - preprocessor vs working directory vs runtime directory, relative vs absolute
 - Limiting symbols in global scope (everything is global by default)
 - Debugging basics: embedding symbols, using breakpoints, watching variables, changing memory, etc
 - Enabling useful compiler warnings

**I/O, OS, code scope**
 - Generic types vs fixed-size types
 - Manual memory management
 - `<iostream>`, `std::getline`, `std::string`/`std::string_view`, `std::atoi`, etc
 - `<fstream>`, `<random>`, `std::stringstream`, `std::vector`, `std::unordered_map`, `std::unordered_set`, `std::list`, `std::deque`, `std::filesystem`, `std::bitset`

**Advanced STL**
 - Move semantics
 - Iterators
 - `<algorithm>` (find, find_if, for_each, any_of, all_of, rotate, transform)
 - Functors and lambdas (immediate vs deferred, lifetime issues with captures)
 - `<functional>`
 - `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`

**Game Fundamentals**
 - Measuring time
 - Input state machines
 - Lifetime-aware Callbacks/Delegates
 - Numerical integration via `deltaTime`
 - Linear interpolation
 - Event loops / game loops
 - Update method
 - Graphics APIs, introduction to OpenGL

**3D Maths**
 - Vectors: geometric interpretation, algebraic operations, multiplications
 - Matrices: geometric interpretation, algebraic operations, multiplications, transformations, translations, rotations, scaling, etc
 - Quaternions: geometric interpretation, algebraic operations, multiplications, interpolations, conversions to angles/matrices
 - Cominbing matrix transformations
 - Model View Matrix pipeline
 - Simple problems in 3D space
