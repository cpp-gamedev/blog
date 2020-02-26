---
title: February Challenges
cover: ./Cpp-Image.png
date: 2020-02-01
description: C++ learning and warm ups
categories: ['posts']
tags: ['challenges']
---

### Number of words in a string

Mini exercise: count the number of words in a string.
**bonus points**: count the number of characters in each word
_Note: you can do both in one pass_

<p>&nbsp;</p>

### Short Exercise:

1. Create a function to "bisect" a string given a delimiter: `"id=fubar", '='` should return `id` and `fubar`.
   Prototype:

```cpp
std::pair<std::string, std::string>
bisect(std::string_view str, char delim);
```

- `pair` can be found in `<utility>`.
- `string_view` instead of `string const&` allows for use of string literals / C strings without suffering a pointless `string` allocation.

2. Create a function to read the contents of a file at a given path into a vector of strings, each representing a line.

```
// File:
tag=fubar
xp=523

// Result:
{"tag=fubar", "xp=523"}
```

Prototype:

```cpp
std::vector<std::string> readString(std::string_view path);
```

3. Use the above functions to read a configuration file into the fields of a struct.
   Eg:

```cpp
struct SaveData
{
  std::string username;
  s32 windowWidth;
  s32 windowHeight;
  bool bFullscreen;
  bool bVsync;
};
```
