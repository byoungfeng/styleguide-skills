---
name: google-style-cpp
description: Google C++ Style Guide — naming, formatting, headers, scoping, classes, and language rules. Use when writing, reviewing, or refactoring C++ code to apply Google's coding standards. Triggers on any C++ coding task.
---

# Google C++ Style Guide — Quick Reference

## Naming Conventions

| Category | Convention | Examples |
|---|---|---|
| File names | `lowercase_underscores` | `file_util.cpp`, `my_class.h` |
| Type names | `PascalCase` | `MyClass`, `HttpRequest`, `FileUtil` |
| Variable (local/param) | `snake_case` | `my_variable`, `file_count` |
| Class data members | `snake_case_` (trailing underscore) | `file_count_`, `name_` |
| Struct data members | `snake_case` (no trailing underscore) | `file_count`, `name` |
| Constants | `kPascalCase` (k prefix) | `kMaxSize`, `kBufferSize` |
| Function names | `PascalCase` | `GetName()`, `IsValid()` |
| Macro names | `UPPER_SNAKE_CASE` | `MY_MACRO`, `LOG_ERROR()` |
| Namespace names | `snake_case` | `my_namespace`, `file_util` |
| Enumerator names | `kPascalCase` (k prefix) | `kOkStatus`, `kNotFound` |
| Template parameters | Follow their category (type → PascalCase, value → snake_case) | `template <typename T>`, `template <int kLimit>` |

## Formatting

- **Line length**: 80 characters max
- **Indentation**: 2 spaces, no tabs
- **Brace style**: K&R (opening brace on same line, except functions/namespaces/classes)
- **Spaces**: Around operators, after keywords (`if`, `while`), inside braces for `{}` init
- **Include order** (groups separated by blank line):
  1. Related header (`.h` for `.cpp` file)
  2. C standard library headers
  3. C++ standard library headers
  4. Other libraries' headers
  5. Project headers (alphabetical within groups)
- **Pointer/reference**: `*` and `&` adjacent to type (`int* p`) or name (`int *p`) — choose one, be consistent

## Header Files

- **#define guards**: `PROJECT_PATH_FILE_H_` — e.g., `MY_PROJECT_SRC_FILE_UTIL_H_`
- **Self-contained**: Headers should compile on their own; include all dependencies
- **Include what you use (IWYU)**: Every symbol must be satisfied by a direct include
- **No forward declarations** of non-owned entities; prefer `#include`
- **Inline functions**: Prefer outside class definition; keep under ~10 lines

## Scoping

- **Namespaces**: Wrap entire file (after includes); no `using namespace` (use `using` declarations instead)
- **Unnamed namespaces**: Preferred over `static` for internal linkage; never in headers
- **Local variables**: Declare in tightest possible scope; initialize on declaration
- **Global variables**: Avoid; use functions, `constexpr`, or singletons (with caution)

## Classes

- **struct vs class**: `struct` for passive objects with public data; `class` for typed objects with invariants
- **Data members**: Trailing underscore for `class` (`count_`), no trailing underscore for `struct` (`count`)
- **Constructors**: `explicit` for single-argument constructors; prefer member initializer lists
- **Defaults**: Use `= default` for destructors/copy/move when compiler-generated is correct; use `= delete` to prohibit
- **`override`**: All virtual method overrides must use `override` keyword (no `virtual` redundantly)
- **Inheritance**: Use `public` inheritance only; prefer composition; mark base destructors `virtual`
- **Access control**: Data members should typically be `private` (use `struct` for public data)

## Language Rules

- **C++ version**: Target C++20, use features responsibly
- **Exceptions**: Allowed (but forbidden on Android); do not add `throw` annotations
- **RTTI**: Avoid (`typeid`, `dynamic_cast`); use virtual functions instead
- **Casting**: Use C++ casts (`static_cast`, `const_cast`, `reinterpret_cast`, `dynamic_cast`); never C-style casts
- **Streams**: Avoid where performance matters; prefer `absl` or `std::string` APIs
- **Preincrement**: Prefer `++i` over `i++` for iterators (no difference for builtins)
- **`const`/`constexpr`**: Use `const` for immutable variables; use `constexpr` for compile-time constants
- **Integer types**: Prefer fixed-width (`int32_t`, `int64_t`, `uint32_t`) over `int`/`long`
- **`auto`**: Use for type deduction when it improves readability; never for return types in public API
- **No VLA**: Use `std::vector` or `std::array` instead of variable-length arrays
- **No `friend`**: Unless absolutely necessary (prefer public accessors)
- **Macros**: Last resort only — prefer `constexpr`, inline functions, templates, enums
- **Braced init lists**: Prefer `std::vector<int> v = {1, 2, 3}`; beware of narrowing
- **Lambdas**: Prefer `[&]`/`[=]` by default; capture individual variables for clarity; avoid default capture in public APIs
- **`std::move`**: Use to enable move semantics; don't use on `const` objects
- **`noexcept`**: Use to enable optimizations when function will never throw
- **Threading**: Use standard threading primitives; avoid raw platform APIs

---

Full source: `G:\styleguide\cppguide.html`
