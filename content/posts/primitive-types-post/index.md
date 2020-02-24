---
title: Primitive Types
cover: ./primitive-programmer.jpg
date: 2020-02-25
decsription: info about generic and fixed size types
tags: ['post']
---

## Primitive Types
C/C++ being rather old languages, the width of types like `int`, `long`, etc have changed over time, from 16 bit CPUs to 32 bit and now 64 bit ones. This keeps programs using such types generic enough to be used on all these target CPU architectures.

However, type sizes are *extremely crucial* in game programming, and we **will never** use generic types. Fixed size types are implementation/compiler defined in a header called `cstdint`, like `uint32_t` for unsigned 32 bit integer, `int64_t` for signed 64 bit integer, etc. Always use these types; create a header that aliases them to `u32` and `s64` / `i64` etc, for shorter and consistent type names.

Useful aliases:
```cpp
using u8 = std::uint8_t;
using s8 = std::int8_t;
using u16 = std::uint16_t;
using s16 = std::int16_t;
using u32 = std::uint32_t;
using u64 = std::uint64_t;
using s32 = std::int32_t;
using s64 = std::int64_t;
using f32 = float;
using f64 = double;
using size_t = std::size_t;
```

