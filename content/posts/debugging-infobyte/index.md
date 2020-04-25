---
title: Debugging
date: 2020-04-24
description: C++ build configurations and debugging
categories: ['posts']
tags: ['infobytes', 'c++']
---

### C++ Memory Model

Unlike higher level languages, the programmer is responsible for _all_ memory management in C/C++. Automatic/stack-allocated memory is automatically cleaned up when it goes out of scope (hence `automatic storage duration`) and should always be preferred, even for managing heap memory (smart pointers). This is the basis of RAII.

A definition like `char x = 'a';` will result in the compiler storing enough space (1 byte) to hold one `char`. Thus the size of the entire type must be known at compile/link time, and the compiler will use this type information to restrict operations on variables/data. This metadata is also used at runtime for RTTI (if enabled), eg `dynamic_cast`. This makes C++ a statically typed language.

C++ has a very strong notion of **scope**, the most common tool to work with them being braces (`{}`): all variables defined within such a scope will be destroyed after the scope ends. Apart from their appearance in functions, `struct`s, `class`es, and `namespace`s, you can introduce an unnamed `{}` scope block almost anywhere you like.

```cpp
{
    Foo foo;    // allocated on the stack, no heap/malloc involved
}   // foo destroyed here
```

When the lifetime of an object is indeterminate at compile time, scope cannot be used to clean it up, and it needs to be allocated on the heap. Traditionally this was done via the `new` operator, which takes in a type (and arguments), allocates sufficient memory for it on the heap (via `malloc`), calls the (appropriate) constructor, and returns a pointer to this object. Such a pointer needs to have operator `delete` invoked on it once its done, which calls the destructor and then releases the associated memory (via `free`). This is not required/recommended anymore; instead we model _object ownership_ and use smart pointers as member variables of owning objects instead.

```cpp
{
    Foo* foo = new Foo;    // allocated on the heap, must be manually deleted
}   // memory leak, pointer to heap allocated memory is lost
```

Storing the pointer in a "higher" scope seems to solve the problem, but introduces another one:
```cpp
Foo* foo = nullptr;
{
    foo = new Foo;
    // operations
}
delete foo; // if an operation throws/returns before this line, there's now a leak
```

C++11 introduced smart pointers, which live on the stack and manage heap memory of any type you provide. Their destructors take care of deleting the allocated pointers. By default, always use `std::unique_ptr`:
```cpp
{
    auto foo = std::make_unique<Foo>();
}   // foo (unique_ptr<Foo>) is a stack object, so it gets destroyed here / on return / on throw / etc (as soon as
    // this scope ends); unique_ptr<Foo>'s destructor then calls delete on the underlying pointer
```

Modern C++ thus warrants designing dynamic memory allocation around ownership models.

A few important consequences of all this:
- A called function's parameters are evaluated and memory for its arguments allocated (on the function stack frame) before it is invoked
- A function's return value can often be _elided_, ie, a copy avoided by emplacing the result directly in lhs memory; this is even more efficient than move semantics, and is called **Return Value Optimisation**
- Arrays live on _the stack!_ Be careful not to overflow due to huge arrays (or endless recursion)
- Using modern C++ you should not need to deal with owning pointers (via `new`) _at all_
- A common misconception is that now "raw pointers are bad"; this is untrue: raw _owning_ pointers are bad, raw _observational_ pointers are still exceptionally "good"
