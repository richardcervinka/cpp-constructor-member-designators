# Proposal for Constructor Member Designators in C++

**Author:** Richard Červinka

**Status:** Draft

**Date:** 2026‑03‑20

## Introduction

This paper proposes a minimal syntactic extension that allows constructors to explicitly refer to non‑static data members in the member‑initializer list using a member designator of the form *.\<member\>*. The feature introduces no new keywords, no changes to overload resolution, no changes to initialization order, and no interaction with C++20 aggregate designated initializers (although the syntax is inspired by them). Its purpose is to eliminate a long‑standing practical issue: parameters and members cannot share names in constructor initializer lists without naming conventions, because x{x} becomes ambiguous.

## Motivation

Today, constructors can directly write this ambigious form:

```c++
Constructor(int value) : value{value} {}
```

This forces widespread and inconsistent naming conventions such as:

- m_x (member prefix)
- x_ (member suffix)
- xArg, xInit, etc.
- Short or cryptic parameter names

These forced conventions reduce clarity and readability. The core reason is that the member‑initializer list lacks any syntax to explicitly say “this identifier is a member”. The grammar does not permit a qualified identifiers such as *this->member* in a member‑initializer.

The proposed design introduces explicit member designators in constructor initializer lists without altering any existing semantics. Initialization order, behavior, lookup rules, and overload resolution all remain unchanged.

The proposed syntax allows writing:

```c++
Constructor(int value) : .value{value} {}
```

This provides a clean, unambiguous way to refer to members of the object under construction, without requiring naming conventions.

## Scope

This proposal applies only to constructor member‑initializer lists.

## Non‑Goals

The feature does not:

- Modify overload resolution (constructors are selected exactly as today)
- Change initialization order (members are initialized in declaration order)
- Extend C++20 designated initializers (which remain aggregate‑only)
- Introduce named function arguments
- Introduce any new keywords
- Alter name lookup outside initializer lists

## Basic Syntax

The proposal allows a leading . in constructor initializer lists to designate a member of the object being constructed. This is purely syntactic sugar for the existing form x{x}, y{y}, but unambiguous and free from naming constraints.

Syntax:

```
: .member { expression }
: .member ( expression )
```

- .member refers to the member member of the object under construction.
- .member always refers to a non‑static data member of the same class.
- .member cannot refer to parameters.
- .member cannot refer to static data members.
- .member cannot refer to inherited members (base subobjects).
- .member cannot refer to anything outside the class.

This eliminates naming conflicts without requiring any naming conventions.

The proposal introduces no new keywords. The token . already exists and is unambiguous in this context.


## Backward compatibility

The syntax is fully backward compatible:
- A leading . is currently invalid in initializer lists.
- No existing code uses this form.
- No macros or libraries are affected.

## Grammar Addition

Extend the grammar for member‑initializer with:

```
member-initializer:
    mem-initializer-id braced-init-list
    . identifier        braced-init-list   // new: constructor member designator
```

## Safety

The semantics mirror existing initializer list rules:
- No method calls allowed.
- No virtual dispatch.
- No access to uninitialized members.
- Only direct member initialization.

## Examples

### Simple class

```c++
class Image
{
public:
    Image(std::string path, int width, int height) :
        .path(std::move(path)),
        .width{width},
        .height{height}
    {}

private:
    std::string path;
    int width;
    int height;
};
```

### Public members + constructor

```c++
struct Vector
{
    Vector() = default;

    Vector(float x, float y) : .x{x}, .y{y} {}

    float x {};
    float y {};
};
```

## Conclusion

Constructor member designators:
- solve a real, universal pain point
- eliminate the need for naming conventions
- improve readability
- align with existing C++20 syntax
- require no new keywords
- are fully backward compatible
- are easy to implement

This proposal provides a small but meaningful improvement to everyday C++ programming.
