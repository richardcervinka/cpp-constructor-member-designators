# Proposal for Constructor Member Designators in C++

**Author:** Richard Červinka

**Status:** Draft

**Date:** 2026‑03‑20

## 1. Introduction

This paper proposes a new syntactic form for member initialization in constructor initializer lists, inspired by C++20 designated initializers. The goal is to eliminate the long‑standing naming conflict between constructor parameters and class members, without introducing new keywords or breaking existing code.

The proposed syntax allows writing:

```c++
Constructor(int value) : .value{value} {}
```

This provides a clean, unambiguous way to refer to members of the object under construction, without requiring naming conventions such as 'm_' prefixes or '_' suffixes.

## 2. Motivation

C++ constructor parameters cannot share names with members, because the initializer list does not allow using this-> or any other explicit qualifier. This forces programmers to adopt naming conventions such as:

- m_value (member prefix)
- value_ (parameter suffix)
- valueInit, valueArg, etc.
- short, semantically poor parameter names (v, x0, rhs)

These conventions are not enforced by the language, vary across codebases, and reduce readability. The root cause is that initializer lists lack a syntax for explicitly referring to members. This proposal introduces such a syntax.

## 3. Proposed feature: Constructor Member Designators

The proposal allows a leading . in constructor initializer lists to designate a member of the object being constructed.

Syntax:
```
: .member { expression }
```

Meaning:

.member refers to the member member of the object under construction.

Example:

```c++
struct Vector
{
    Vector(float x, float y) : .x{x}, .y{y} {}

    float x, y;
};
```

This eliminates naming conflicts without requiring any naming conventions.

## 4. Design considerations

### 4.1 No new keywords

The proposal introduces no new keywords. The token . already exists and is unambiguous in this context.

### 4.2 Consistency with C++20 designated initializers

C++20 introduced:

```c++
Point p{ .x = 1, .y = 2 };
```

Constructor member designators are a natural extension of this syntax.

### 4.3 Backward compatibility

The syntax is fully backward compatible:
- A leading . is currently invalid in initializer lists.
- No existing code uses this form.
- No macros or libraries are affected.

### 4.4 Safety

The semantics mirror existing initializer list rules:
- No method calls allowed.
- No virtual dispatch.
- No access to uninitialized members.
- Only direct member initialization.

## 5. Examples

### 5.1 Simple class

```c++
class Image
{
public:
    Image(std::string path, int width, int height) :
        .path{std::move(path)},
        .width{width},
        .height{height}
    {}

private:
    std::string path;
    int width;
    int height;
};
```

### 5.2 Public members + constructor

```c++
struct Vector
{
    Vector() = default;

    Vector(float x, float y) : .x{x}, .y{y} {}

    float x {};
    float y {};
};
```

## 6. Implementation considerations

This feature is syntactic sugar. It requires:
- a parser change
- no ABI changes
- no changes to object layout
- no changes to overload resolution
- no changes to name lookup rules outside initializer lists

## 7. Conclusion

Constructor member designators:
- solve a real, universal pain point
- eliminate the need for naming conventions
- improve readability
- align with existing C++20 syntax
- require no new keywords
- are fully backward compatible
- are easy to implement

This proposal provides a small but meaningful improvement to everyday C++ programming.
