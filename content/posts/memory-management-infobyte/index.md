---
title: Memory Management
date: 2020-02-26
description: C++ memory management
categories: ['posts']
tags: ['infobytes']
---

### Memory management

Unlike higher level languages, the programmer is responsible for _all_ memory management in C/C++. Automatic/stack-allocated memory is automatically cleaned up when it goes out of scope (hence `automatic storage duration`) and should always be preferred, even for managing heap memory (smart pointers). This is the basis of RAII.

```cpp
{
    Foo foo;
}   // foo destroyed here
```

When the lifetime of an object is indeterminate at compile time, scope cannot be used to clean it up, and it needs to be allocated on the heap. Traditionally this was done via the `new` operator, which takes in a type (and arguments), allocates sufficient memory for it on the heap (via `malloc`), calls the (appropriate) constructor, and returns a pointer to this object. Such a pointer needs to have operator `delete` invoked on it once its done, which calls the destructor and then releases the associated memory (via `free`). This is not required/recommended anymore; instead we model _object ownership_ and use smart pointers as member variables of owning objects instead.

```cpp
{
    Foo* foo = new Foo;
}   // memory leak, pointer to heap allocated memory is lost

{
    Foo* foo = new Foo;
    // operations
    delete foo; // if an operation throws/returns before this line, there's now a leak
}

{
    auto foo = std::make_unique<Foo>();
}   // foo is a stack object, so it gets destroyed here / on return / on throw / etc (as soon as
    // this scope ends); unique_ptr<Foo>'s destructor then calls delete on the underlying pointer
```

Modern C++ thus warrants designing dynamic memory allocation around ownership models.
