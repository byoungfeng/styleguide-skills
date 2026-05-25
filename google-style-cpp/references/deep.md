# Google C++ Style Guide — Deep Reference

---

## 1. Header Files

### 1.1 Self-Contained Headers

Every header file must compile on its own. When a `.cpp` file `#include`s a header, it should not need to include anything else beforehand to compile correctly. This means each header must directly include everything it uses.

```cpp
// Good — MyClass.h includes everything it needs
#include <string>
#include <vector>

class MyClass {
 public:
  explicit MyClass(const std::string& name);
  void AddValue(int value);

 private:
  std::string name_;
  std::vector<int> values_;
};
```

```cpp
// Bad — relies on caller to include <string> first
#include <vector>

class MyClass {
 public:
  explicit MyClass(const std::string& name);  // Error! std::string not declared
  void AddValue(int value);

 private:
  std::string name_;
  std::vector<int> values_;
};
```

To verify self-containment, the related `.cpp` file should include the `.h` before anything else (see include order below).

### 1.2 #define Guard Format

Use the `PROJECT_PATH_FILE_H_` naming convention. The guard name derives from the file path within the project.

```
// Project structure: my_project/src/base/file_util.h
#ifndef MY_PROJECT_SRC_BASE_FILE_UTIL_H_
#define MY_PROJECT_SRC_BASE_FILE_UTIL_H_

// ... contents ...

#endif  // MY_PROJECT_SRC_BASE_FILE_UTIL_H_
```

Rules:
- Full path from project root, normalized to uppercase
- Replace `/`, `.`, `-` with `_`
- Append a trailing `_`
- Do not use `#pragma once` — `#define` guards are required for portability

```
// Examples:
//   src/core/math.h            →  CORE_MATH_H_
//   my_lib/include/foo.h       →  MY_LIB_INCLUDE_FOO_H_
//   base/strings/string_util.h →  BASE_STRINGS_STRING_UTIL_H_
```

### 1.3 Include What You Use (IWYU)

Every symbol used in a file must be provided by an explicit `#include` in that file. Never rely on transitive includes — they are an implementation detail of the headers you include.

```cpp
// foo.h
#ifndef FOO_H_
#define FOO_H_

#include <string>

class Foo {
 public:
  std::string GetName() const;  // <string> included directly
};

#endif  // FOO_H_
```

```cpp
// foo.cpp
#include "foo.h"

#include <algorithm>
#include <vector>

#include "bar.h"
#include "baz.h"

// Even though <vector> might be transitively included by bar.h, we include it
// directly because we use std::vector here.
```

Exception: When moving code and the include is clearly redundant *and* removing it doesn't break the build, you may remove it. When in doubt, keep the include.

Include order (each group separated by a blank line, sorted alphabetically within groups):

1. The `.h` that corresponds to this `.cpp` (guarantees self-containment)
2. C standard library headers (`<cstdlib>`, `<cstdio>`)
3. C++ standard library headers (`<string>`, `<vector>`)
4. Other libraries' headers (e.g., `gtest/gtest.h`, `absl/status/status.h`)
5. Project headers (`"base/logging.h"`, `"my_project/core/math.h"`)

```cpp
// my_project/src/core/foo.cpp

// 1. Related header
#include "my_project/core/foo.h"

// 2. C headers
#include <cstdint>
#include <cstdlib>

// 3. C++ headers
#include <algorithm>
#include <memory>
#include <string>
#include <vector>

// 4. Other libraries
#include "absl/status/status.h"
#include "gtest/gtest.h"

// 5. Project headers
#include "my_project/base/logging.h"
#include "my_project/core/math.h"
```

Conditional includes (platform-specific) should be kept minimal and placed after the main blocks with a comment.

### 1.4 Forward Declarations

Prefer `#include` over forward declarations. Forward declarations are permitted but discouraged because they:

**Pros:**
- Reduce compilation time (fewer files need recompilation)
- Break dependency cycles (though cycles indicate poor design)

**Cons:**
- Hide the dependency — users may not know what they need to include
- Library authors may change APIs; a forward declaration can silently break
- Templates, enums, and nested classes cannot be forward-declared
- `std` namespace entities should never be forward-declared (undefined behavior)

When to use forward declarations:
- Only for types you do **not** own (though `#include` is preferred here too)
- Never for the C++ standard library
- Never when you need the full definition (size, members, methods)

```cpp
// Forward declaration — use with caution
class Bar;  // Only if Bar is a class you don't own and cannot include

class Foo {
 public:
  void Process(Bar* bar);  // Pointer or reference only — incomplete type is fine

 private:
  Bar* bar_;
};
```

### 1.5 Inline Functions

Define functions `inline` only when they are small (typically 10 lines or fewer). Do not put large functions in header files just to make them inline.

```cpp
// Good — small accessor, appropriate for inlining
inline int GetCount() const { return count_; }
```

```cpp
// Bad — too large for inlining, belongs in .cpp
inline int ComputeHash(const std::string& input) {
  int hash = 0;
  for (char c : input) {
    hash = hash * 31 + static_cast<int>(c);
  }
  return hash;
}
```

Prefer defining inline functions outside the class declaration:

```cpp
class MyClass {
 public:
  int GetCount() const;  // Declaration only
};

// Outside class — still inline
inline int MyClass::GetCount() const { return count_; }
```

Virtual functions and recursive functions should never be `inline` (the compiler cannot inline them).

---

## 2. Scoping

### 2.1 Namespace Rules

- Wrap all code (after includes) in a project-level namespace
- Do not indent namespace contents

```cpp
namespace my_project {
namespace core {

class Foo {
 public:
  void Bar();
};

}  // namespace core
}  // namespace my_project
```

- **Never use `using namespace`** (neither in headers nor .cpp files). It pollutes the namespace and causes subtle bugs.
- Prefer `using` declarations for specific names:

```cpp
// Good
using std::string;
using std::vector;
```

```cpp
// Bad
using namespace std;
```

- `using` declarations are allowed in `.cpp` files and, rarely, in internal helper namespaces within headers.
- Never put `using` directives in header files in the global namespace.

Close nested namespaces with comments:

```cpp
namespace my_project {
namespace core {
namespace internal {
// ...
}  // namespace internal
}  // namespace core
}  // namespace my_project
```

### 2.2 Unnamed Namespaces vs `static`

Prefer unnamed namespaces over `static` for file-scoped declarations. This gives internal linkage to all contained names.

```cpp
// Good — unnamed namespace
namespace {

constexpr int kBufferSize = 4096;

int InternalHelperFunction(int x) {
  return x * 2;
}

}  // namespace
```

```cpp
// Acceptable but less idiomatic
static constexpr int kBufferSize = 4096;

static int InternalHelperFunction(int x) {
  return x * 2;
}
```

**Important**: Unnamed namespaces in headers bind the contents to each translation unit separately, leading to code bloat. If you need internal linkage in a header, use a named `internal` namespace instead and document that the contents are not part of the public API.

### 2.3 Local Variables

- Declare variables in the narrowest possible scope
- Initialize on declaration — never declare without initializing

```cpp
// Good
std::string name = GetName();
int count = 0;
```

```cpp
// Bad — uninitialized
std::string name;
int count;
// ... later ...
name = GetName();
count = 0;
```

- Move loop-scoped variables inside the loop:

```cpp
// Good
for (int i = 0; i < 100; ++i) {
  std::string item = GetItem(i);
  Process(item);
}
```

```cpp
// Bad — variable lives longer than needed
std::string item;
for (int i = 0; i < 100; ++i) {
  item = GetItem(i);
  Process(item);
}
```

### 2.4 Nonmember, Static Member, and Global Functions

- Prefer nonmember functions in a namespace over static member functions
- Prefer static member functions (in a class) over global functions
- Avoid global non-`const` variables entirely
- Group related functions into a namespace

```cpp
namespace my_project {
namespace file_util {

// Prefer this — namespace-scoped nonmember functions
std::string ReadFile(const std::string& path);
bool WriteFile(const std::string& path, const std::string& content);

}  // namespace file_util
}  // namespace my_project
```

```cpp
// Acceptable when the function must be associated with a class
class FileManager {
 public:
  static std::string ReadFile(const std::string& path);
  static bool WriteFile(const std::string& path, const std::string& content);
};
```

Global variables (non-const) are strongly discouraged. Use `constexpr` constants, or for mutable global state, use a function-local static:

```cpp
// Acceptable — function-local static (thread-safe in C++11+)
const std::string& GetConfigPath() {
  static const std::string* kPath = new std::string("/etc/my_app/config.ini");
  return *kPath;
}
```

---

## 3. Classes

### 3.1 Struct vs Class

Use `struct` for passive objects that carry data with no invariants. Use `class` for objects that maintain invariants and encapsulate behavior.

```cpp
// struct — public data by default, no invariants
struct Point {
  double x;
  double y;
  double z;
};

// class — private data by default, maintains invariants
class Temperature {
 public:
  explicit Temperature(double celsius) : celsius_(celsius) {}

  double ToFahrenheit() const { return celsius_ * 9.0 / 5.0 + 32.0; }
  double ToKelvin() const { return celsius_ + 273.15; }

 private:
  double celsius_;  // Invariant: >= -273.15
};
```

Key distinction for naming:
- **Class data members**: `snake_case_` (trailing underscore)
- **Struct data members**: `snake_case` (no trailing underscore)

If a struct has any private/protected members or member functions with logic, it should probably be a `class`.

### 3.2 Inheritance

- Prefer composition over inheritance
- Use `public` inheritance only (to model "is-a" relationships); never `protected` or `private` inheritance
- Base class destructors must be `virtual` if the class is intended as a base

```cpp
class Base {
 public:
  virtual ~Base() = default;
  virtual void DoSomething() = 0;
};

class Derived : public Base {
 public:
  void DoSomething() override { /* ... */ }
};
```

- Multiple inheritance is allowed but rare; prefer interfaces (pure abstract classes)
- If using multiple inheritance, at most one base should have non-static data members

### 3.3 Operator Overloading

Overload operators only when the meaning is obvious and consistent with built-in expectations.

```cpp
class Vector2 {
 public:
  Vector2 operator+(const Vector2& other) const {
    return Vector2{x_ + other.x_, y_ + other.y_};
  }

  bool operator==(const Vector2& other) const {
    return x_ == other.x_ && y_ == other.y_;
  }

 private:
  double x_;
  double y_;
};
```

Do not overload `operator&&`, `operator||`, `operator,`, or `operator&` (address-of) — these break short-circuit evaluation rules and confuse readers.

Define overloaded operators as nonmember functions when they can be:

```cpp
Vector2 operator*(double scalar, const Vector2& v) {
  return Vector2{v.X() * scalar, v.Y() * scalar};
}
```

### 3.4 Access Control

- Data members should typically be `private`
- Use `public:` / `protected:` / `private:` without extra indentation (or 1-space indent — be consistent)
- Expose data through accessors (Getters) and mutators (Setters) as needed

```cpp
class Account {
 public:
  explicit Account(std::string owner) : owner_(std::move(owner)) {}

  const std::string& GetOwner() const { return owner_; }
  double GetBalance() const { return balance_; }

  void Deposit(double amount);

 private:
  std::string owner_;
  double balance_ = 0.0;
};
```

Access sections should appear in order: `public:`, then `protected:`, then `private:`.

### 3.5 Explicit Constructors

Use the `explicit` keyword for single-argument constructors (including constructors with default arguments) to prevent implicit conversions.

```cpp
// Good — prevents Foo f = 42;
class Foo {
 public:
  explicit Foo(int value);
};
```

```cpp
// Bad — allows Foo f = 42; which is surprising
class Foo {
 public:
  Foo(int value);
};
```

Exception: Copy and move constructors should not be `explicit`.

Implicit conversions are acceptable only for types that are genuinely interchangeable (e.g., `std::string_view` → `std::string` is not implicit because it requires allocation):

```cpp
// Acceptable — wrapping a numeric type with clear interchangeability
class WrappedInt {
 public:
  // Intentionally not explicit — WrappedInt is "transparent" over int
  WrappedInt(int value) : value_(value) {}
  int GetValue() const { return value_; }
 private:
  int value_;
};
```

### 3.6 Declaring Data Members

- Always use trailing underscore for class data members (`member_name_`):
- Always initialize data members (in-class initializers or constructor init list)
- Struct members get no trailing underscore:

```cpp
class HttpRequest {
 public:
  HttpRequest() : method_("GET") {}

 private:
  std::string method_;
  std::string url_;
  int timeout_ms_ = 30000;
};

struct HttpConfig {
  std::string proxy;
  int timeout_ms;
  bool retry_on_failure;
};
```

---

## 4. Formatting

### 4.1 Line Length Edge Cases

Maximum line length is **80 characters**. Exceptions:
- Long strings/string literals (break with concatenation or `string_view`)
- Long includes (they cannot be broken)
- Long URLs in comments (include as-is)
- Long raw string literals

```cpp
// Breaking a long line — align with the opening parenthesis
bool result = SomeLongFunctionName(
    arg1, arg2, arg3, arg4);

// String literal continuation
std::string message =
    "This is a very long string that exceeds 80 characters and "
    "needs to be split across two lines.";

// Lambda continuation
auto lambda = [this](const std::string& long_param_name,
                     int another_long_param) {
  DoSomething(long_param_name, another_long_param);
};
```

### 4.2 Brace Styles

Google C++ Style uses **K&R (Kernighan & Ritchie)** brace style:

```cpp
// Functions, namespaces, classes — brace on new line
int SomeFunction(int x)
{
  if (x > 0) {
    DoSomething();
  } else {
    DoSomethingElse();
  }
}
```

```cpp
// Control flow — brace on same line
if (condition) {
  // ...
} else if (other_condition) {
  // ...
} else {
  // ...
}
```

```cpp
// Loops — brace on same line
for (int i = 0; i < 10; ++i) {
  Process(i);
}

while (condition) {
  // ...
}

do {
  // ...
} while (condition);
```

```cpp
// Switch — brace on same line, cases indented
switch (value) {
  case kOptionOne:
    HandleOne();
    break;
  case kOptionTwo:
    HandleTwo();
    break;
  default:
    HandleDefault();
    break;
}
```

Empty braces may be placed on the same line:

```cpp
class Foo {};  // Empty class body
if (condition) {}  // Empty statement
```

### 4.3 Spaces vs Tabs

- **2-space indent** throughout
- **No tabs** — spaces only
- Most editors can be configured to insert spaces when Tab is pressed

```cpp
namespace my_project {

class Sample {
 public:
  void DoSomething() {
    if (condition) {
      // Indented by 4 spaces inside if block
      Process();
    }
    // Indented by 2 spaces inside function body
  }

 private:
  // Indented by 2 spaces
  int member_ = 0;
};

}  // namespace my_project
```

### 4.4 Function Declarations and Wrapping

When a function declaration exceeds 80 characters, wrap:

```cpp
// Leading decorators on separate lines
void LongFunctionNameThatExceedsEightyCharacters(
    const std::string& first_param,
    const std::string& second_param,
    int third_param,
    double fourth_param);
```

```cpp
// Return type on its own line for long signatures
bool IsAVeryVeryLongFunctionNameThatExceedsEightyCharacters(
    const std::string& param1, const std::string& param2);
```

```cpp
// Member functions with qualifiers
bool MyClass::AVeryLongFunctionNameThatExceeds80Characters(
    const std::string& param1, const std::string& param2) const {
  // ...
}
```

Alignment: Parameters should align with the opening parenthesis or use a 4-space indent.

```cpp
// Align with opening paren
ReturnType LongFunctionName(TypeName param1,
                            TypeName param2,
                            TypeName param3);

// 4-space indent when alignment would be awkward
ReturnType ExtremelyLongFunctionNameThatWraps(
    TypeName param1, TypeName param2, TypeName param3);
```

### 4.5 Lambda Formatting

```cpp
// Simple lambda — same line
auto lambda = [](int x) { return x * 2; };

// Multiple lines — brace on same line
auto lambda = [this](const std::string& name) {
  std::string result = Process(name);
  return result;
};

// Lambda as argument — break before lambda if line is long
std::sort(v.begin(), v.end(),
          [](const std::string& a, const std::string& b) {
            return a.size() < b.size();
          });
```

Captures:
- `[&]` — capture by reference (default when lambda outlives not a concern)
- `[=]` — capture by value (default when lambda might outlive scope)
- `[this]` — capture `this` explicitly
- `[x, &y]` — explicit captures for clarity

Avoid default captures in public APIs as they make the capture set unclear.

### 4.6 Boolean Expressions

When a boolean expression exceeds 80 characters, break after operators:

```cpp
// Good — operators at beginning of line
if (some_very_long_condition_variable_1
    && some_very_long_condition_variable_2
    && some_very_long_condition_variable_3) {
  DoSomething();
}
```

```cpp
// Also acceptable — indented continuation
if (some_very_long_condition_variable_1 &&
        some_very_long_condition_variable_2 &&
        some_very_long_condition_variable_3) {
  DoSomething();
}
```

### 4.7 Preprocessor Directives Formatting

Preprocessor directives start at column 0 (no indentation), even inside blocks:

```cpp
void Foo() {
#if defined(OS_WIN)
  WindowsSpecificCode();
#elif defined(OS_LINUX)
  LinuxSpecificCode();
#else
  OtherCode();
#endif
}
```

For long macro definitions, use `\` continuation:

```cpp
#define MY_MACRO(x, y, z)           \
  do {                              \
    auto value = (x) + (y);         \
    if (value > (z)) {              \
      Process(value);               \
    }                               \
  } while (false)
```

---

## 5. Naming

### 5.1 General Naming Rules

Names should be descriptive; avoid abbreviations unless they are extremely common.

```cpp
// Good
int num_errors;          // Not int n;
std::string table_name;  // Not std::string tn;
```

```cpp
// Bad — cryptic abbreviations
int n;                          // What is n?
std::string tn;                 // table_name? temp_name?
DoubleLinkedNode* dlnode;       // Just say node or list_node
```

Common acceptable abbreviations: `tmp`, `i`/`j`/`k` (loop counters), `ret` (return value), `num` (count), `init` (initialization), `src`/`dst` (source/destination).

### 5.2 File Names

- **Lowercase with underscores** (e.g., `file_util.cpp`, `http_request.h`)
- If no underscore separation works (one-word name), lowercase is fine (`main.cpp`, `parser.h`)
- Use `.h` extension for headers, `.cpp` for implementation
- C headers intended for use with C++ should use `.h` and be wrapped in `extern "C"` if needed
- Avoid overly generic names like `utils.h`

```cpp
// Good file names
//   http_request.h
//   http_request.cpp
//   string_util.h
//   main.cpp
//   BUILD

// Bad file names
//   HTTPRequest.h    — uppercase
//   httpreq.h         — cryptic
//   Util.h            — overly generic
```

### 5.3 Type Names

- **PascalCase** — starts with uppercase, each word capitalized, no underscores
- Includes classes, structs, enums, typedefs, type aliases, template type parameters

```cpp
// Classes and structs
class HttpRequest { /* ... */ };
struct ConnectionInfo { /* ... */ };

// Enums
enum class StatusCode { kOk, kNotFound };

// Typedefs and aliases
using StringMap = std::map<std::string, std::string>;
typedef std::map<std::string, std::string> StringMapOldStyle;

// Template type parameters
template <typename T>
class Container { /* ... */ };
```

### 5.4 Variable Names

**General variables** (local, function parameters, non-member): `snake_case`

```cpp
std::string file_path;
int line_count;

void ProcessData(const std::string& input_text, int max_lines);
```

**Class data members**: `snake_case_` (trailing underscore)

```cpp
class Account {
 public:
  explicit Account(const std::string& owner) : owner_(owner) {}

 private:
  std::string owner_;
  double balance_ = 0.0;
};
```

**Struct data members**: `snake_case` (no trailing underscore)

```cpp
struct Address {
  std::string street;
  std::string city;
  std::string zip_code;
};
```

**Static members follow class/struct rules** — no special prefix like `s_` or `g_`:

```cpp
class Config {
 public:
  static constexpr int kDefaultPort = 8080;

 private:
  static std::string config_path_;
};
```

### 5.5 Constant Names

- **`kPascalCase`** — `k` prefix followed by PascalCase
- Applies to all constants, including `constexpr`, `const`, and enum values

```cpp
// Global/class constants
constexpr int kMaxFileSize = 1048576;
const int kBufferSize = 4096;
constexpr double kEpsilon = 1e-9;

// Class-scoped constants
class Math {
 public:
  static constexpr double kPi = 3.141592653589793;
  static constexpr int kMaxIterations = 1000;
};

// Enum values
enum class ErrorCode {
  kSuccess = 0,
  kNotFound = 1,
  kPermissionDenied = 2,
};
```

Note: `const` local variables use `snake_case` (they are still variables):

```cpp
void Foo() {
  const int max_attempts = 3;  // Local const — snake_case is fine
  for (int i = 0; i < max_attempts; ++i) {
    // ...
  }
}
```

### 5.6 Function Names

- **PascalCase** — starts with uppercase, each word capitalized
- Names should be verbs or verb phrases

```cpp
// Accessors
std::string GetName() const;
void SetName(const std::string& name);

// Boolean queries
bool IsValid() const;
bool HasData() const;

// Actions
void ProcessRequest();
void WriteToFile(const std::string& path);

// Conversion
int ToInt() const;
std::string ToString() const;
```

Return value accessors should omit `Get_` when possible:

```cpp
class Foo {
 public:
  int count() const { return count_; }       // Prefer this
  // int GetCount() const { return count_; }  // Also acceptable

 private:
  int count_ = 0;
};
```

Non-constant accessors (returning a non-const reference/pointer) should be named to avoid confusion with const accessors:

```cpp
class Foo {
 public:
  const std::string& name() const { return name_; }
  std::string* mutable_name() { return &name_; }

 private:
  std::string name_;
};
```

### 5.7 Namespace Names

- **`snake_case`**
- Short, descriptive, no abbreviations
- Top-level namespace should be the project name

```cpp
namespace my_project {
namespace internal {
namespace file_util {
// ...
}  // namespace file_util
}  // namespace internal
}  // namespace my_project
```

### 5.8 Enumerator Names

- **`kPascalCase`** — same as constants (preferred)
- Or `UPPER_SNAKE_CASE` — but `kPascalCase` is strongly preferred

```cpp
// Preferred — kPascalCase
enum class Color {
  kRed,
  kGreen,
  kBlue,
};

// Acceptable but not preferred
enum class OldStyle {
  RED,
  GREEN,
  BLUE,
};
```

### 5.9 Macro Names

- **`UPPER_SNAKE_CASE`**
- Macros are a last resort; prefer `constexpr`, `inline`, templates, or enums

```cpp
// If you absolutely must use a macro
#define MY_PROJECT_LOG_ERROR(msg) \
  LogError(__FILE__, __LINE__, msg)
```

---

## 6. Comments

### 6.1 Comment Style

- Use `//` for all comments (including multi-line blocks)
- Never use `/* */` (except for Doxygen-style documentation, and even then `//` is preferred)
- Comments should be full sentences with proper capitalization and punctuation

```cpp
// Good — use double-slash for all comments
// This is a multi-line comment block.
// Each line starts with //.

// Bad — avoid C-style block comments
/* This is not the preferred style. */
```

### 6.2 File Comments

Start every file with a license boilerplate (project-dependent). After the license, add a brief description of the file's purpose.

```cpp
// Copyright 2024 Google LLC
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     https://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
//
// -----------------------------------------------------------------------------
// File: http_request.h
// -----------------------------------------------------------------------------
//
// Defines the HttpRequest class used to represent and process HTTP requests.
```

For small/obvious files, a one-line comment suffices after the license.

### 6.3 Class Comments

Document what the class represents, its thread-safety guarantees, and usage example if non-trivial.

```cpp
// Represents an HTTP request from a client. Thread-safe once constructed.
//
// Usage:
//   HttpRequest request;
//   request.SetMethod("GET");
//   request.SetUrl("https://example.com/api");
//   request.Send();
//
// All public methods are thread-safe after construction.
class HttpRequest {
 public:
  HttpRequest();
  ~HttpRequest();

  void SetMethod(const std::string& method);
  void SetUrl(const std::string& url);
  void Send();

 private:
  class Impl;
  std::unique_ptr<Impl> impl_;
};
```

### 6.4 Function Comments

- Declaration comments describe the function's purpose and usage (not how it works)
- Definition comments describe implementation details if non-trivial

```cpp
// Declaration comment (in .h):
//
// Fetches the resource at `url` and returns its contents.
// Returns an empty string on failure.
// Thread-safe.
std::string FetchUrl(const std::string& url);

// Definition comment (in .cpp):
//
// Uses libcurl internally. The response buffer is allocated on the heap
// and released when the request completes.
std::string FetchUrl(const std::string& url) {
  // ... implementation ...
}
```

For simple accessors, comments are often unnecessary:

```cpp
int count() const { return count_; }  // No comment needed — obvious
```

### 6.5 Variable Comments

- Comments for class/struct data members describe their meaning and invariants
- Local variable comments are rarely needed (the name should be self-documenting)

```cpp
class NetworkConnection {
 private:
  // The underlying socket file descriptor. -1 means not connected.
  int socket_fd_ = -1;

  // Timeout in milliseconds. Must be > 0.
  int timeout_ms_ = 5000;

  // Number of bytes sent since last reset. Monotonically increasing.
  std::atomic<int64_t> bytes_sent_{0};
};
```

### 6.6 TODO Style

```cpp
// TODO(username): Add retry logic for network failures.
// TODO(b/12345678): Remove this workaround when the bug is fixed.
// TODO(username@domain.com): Migrate to the new API in Q3 2024.
```

The `TODO` should include a reference (name, bug ID, or email) followed by a colon and the action item.

### 6.7 Punctuation and Spelling

- Use proper capitalization and punctuation in comments
- Check spelling
- Use present tense ("Fetches the resource" not "Will fetch the resource")
- Use "Returns" not "Return"
- Be precise: "Returns the number of active connections" not "Returns the count"

---

## 7. Language Rules

### 7.1 C++ Version Policy

- Target **C++20** (the latest C++ standard supported across all target platforms)
- Use C++20 features that are widely supported and improve code quality:
  - Concepts (`template <std::integral T>`)
  - `std::span` (non-owning array views)
  - `std::string_view` and `std::string::starts_with`/`ends_with`
  - `constexpr` improvements (virtual functions, `std::vector`, `std::string`)
  - `std::optional`, `std::variant`
  - Designated initializers
  - Three-way comparison (`<=>`, but use cautiously)
- Avoid experimental or poorly-supported C++20 features

### 7.2 Exceptions

Exceptions are **permitted** but with restrictions:
- Allowed in most code (desktop/server targets)
- **Forbidden on Android** (Android toolchain historically had poor exception support)
- Code that uses exceptions and code that does not should coexist peacefully
- Do not add `throw` or `noexcept` annotations to function declarations (they affect caller code and ABI)
- Use exceptions for truly exceptional conditions, not for normal control flow

```cpp
// Good — throw for exceptional conditions
if (!FileExists(path)) {
  throw std::runtime_error("File not found: " + path);
}

// Bad — using exceptions for control flow
try {
  int value = ConvertToInt(user_input);
} catch (const std::invalid_argument&) {
  // Expected — user input might be invalid
}
```

### 7.3 RTTI (Run-Time Type Information)

- **Avoid RTTI** (`typeid`, `dynamic_cast`)
- Prefer virtual functions to dispatch based on type
- If you must use `dynamic_cast`, ensure the code path is a clear "is-a" check

```cpp
// Good — virtual function dispatch
class Shape {
 public:
  virtual double Area() const = 0;
};

class Circle : public Shape {
 public:
  double Area() const override { return kPi * radius_ * radius_; }
 private:
  double radius_;
};

// Bad — RTTI-based dispatch
void PrintShape(const Shape& shape) {
  if (typeid(shape) == typeid(const Circle&)) {
    // ...
  }
}
```

The one acceptable use of `dynamic_cast` is in test code or extremely rare cases where virtual functions are impossible (e.g., casting from a template base).

### 7.4 Casting

**Never use C-style casts.** Use C++ casts which are explicit and searchable.

```cpp
// Good
float f = 3.14f;
int i = static_cast<int>(f);

const int* const_p = &i;
int* p = const_cast<int*>(const_p);  // Only when you must (rare)

void* raw = reinterpret_cast<void*>(p);  // Only when interfacing with C APIs

// Bad — C-style casts are forbidden
int i = (int)f;
int* p = (int*)const_p;
```

`static_cast` is the default cast. Use `static_cast` to:
- Convert between numeric types (including enum to int)
- Downcast within a class hierarchy (when you know the type is correct)
- Convert `void*` to typed pointer

`reinterpret_cast` is only for:
- Converting between pointer types (e.g., `T*` → `char*`)
- Converting pointer to/from integer type (platform-specific)
- Never use `reinterpret_cast` to bypass type safety in normal code

### 7.5 Streams

- Use streams (`<iostream>`, string streams) for simple formatting
- Avoid streams in performance-critical code (they allocate and dispatch virtually)
- Prefer `absl::StrCat`, `absl::StrFormat`, or `std::format` (C++20) for string assembly

```cpp
// Acceptable for simple output
std::cout << "Result: " << value << std::endl;

// Prefer for string construction (especially in loops)
std::string result = absl::StrCat("Result: ", value);

// C++20 option
std::string result = std::format("Result: {}", value);
```

### 7.6 Preincrement and Predecrement

- Prefer `++i` over `i++` (same for decrement)
- For built-in types, there is no difference
- For iterators and other class types, prefix avoids a copy of the old value

```cpp
// Prefer
for (auto it = v.begin(); it != v.end(); ++it) { /* ... */ }

// Avoid
for (auto it = v.begin(); it != v.end(); it++) { /* ... */ }
```

### 7.7 `const` Usage

Use `const` wherever a value or pointer should not be modified. This includes:
- Function parameters that are not modified
- Member functions that do not modify the object
- Local variables that do not change after initialization

```cpp
// Const parameters
void ProcessData(const std::string& input, std::string* output);

// Const member function
class Foo {
 public:
  int GetCount() const { return count_; }
  void SetCount(int count) { count_ = count; }

 private:
  int count_ = 0;
};

// Const local variable
void Bar() {
  const std::string greeting = BuildGreeting();
  // greeting cannot be modified
}
```

Place `const` before the type: `const int&` not `int const&`.

### 7.8 `constexpr`

Use `constexpr` for:
- True compile-time constants
- Functions that can be evaluated at compile time
- Variables whose value is known at compile time

```cpp
constexpr int kPageSize = 4096;
constexpr double kE = 2.718281828459045;

constexpr int Factorial(int n) {
  return n <= 1 ? 1 : n * Factorial(n - 1);
}

// Usage — computed at compile time
constexpr int kFactorial10 = Factorial(10);
```

In C++20, use `consteval` for functions that *must* be evaluated at compile time (never called at runtime):

```cpp
consteval int Square(int n) { return n * n; }
constexpr int kVal = Square(5);  // OK — compiled
// int x = Square(y);            // Error — must be constant
```

### 7.9 Integer Types

- Prefer fixed-width integer types from `<cstdint>`:
  - `int32_t`, `uint32_t`, `int64_t`, `uint64_t`
  - `int16_t`, `uint16_t`, `int8_t`, `uint8_t` (use with caution — avoid `char` confusion)
- Use `size_t` for sizes/indices (or `int` when size_t's unsigned semantics are undesirable)
- Use `ptrdiff_t` for pointer differences
- Never assume `int` is 32-bit or `long` is 64-bit

```cpp
// Good — explicit width
int32_t file_size = GetFileSize(path);
int64_t total_bytes = ComputeTotal();

// Good — size_t for sizes
for (size_t i = 0; i < v.size(); ++i) {
  // ...
}

// Bad — platform-dependent width
int file_size = GetFileSize(path);  // May overflow on some platforms
```

### 7.10 64-Bit Portability

Be aware of 64-bit portability issues:
- `printf` specifiers: use `PRId64` from `<cinttypes>` for `int64_t`
- Pointer-to-int conversion: use `intptr_t` or `uintptr_t` if needed; never `int` or `long`
- Structure packing: `#pragma pack` is allowed but needs careful review

```cpp
#include <cinttypes>

int64_t value = GetValue();
printf("Value: %" PRId64 "\n", value);  // Portable 64-bit format
```

### 7.11 `auto`

Use `auto` when it improves readability:
- Complex iterator types
- Lambda types (which are unnameable)
- Template expressions
- When the type is obvious from context

```cpp
// Good — iterator types are verbose
auto it = container.find(key);

// Good — lambda type is unnameable
auto lambda = [](int x) { return x * 2; };

// Good — type is obvious from context
auto value = GetValue();  // If GetValue() clearly returns an int

// Good — template expressions
auto result = std::make_unique<MyClass>();

// Bad — obscures readability
auto result = some_function(a, b, c);  // What does it return?

// Bad — auto return in public API
auto GetName() const { return name_; }  // Use std::string explicitly
```

Never use `auto` for function return types in public header files — the return type is part of the API contract.

### 7.12 Braced Init Lists

Use braced initializer lists for aggregate types and containers. Be aware of narrowing conversions:

```cpp
// Aggregates
Point p = {1.0, 2.0, 3.0};

// Containers
std::vector<int> numbers = {1, 2, 3, 4, 5};
std::map<std::string, int> scores = {{"alice", 10}, {"bob", 20}};

// Narrowing — brace init prevents this
int x = 3.14;         // OK — x becomes 3
int y = {3.14};       // Compiler error — narrowing conversion
```

When braced init lists interact with `auto`, the type deduced is `std::initializer_list`:

```cpp
auto v = {1, 2, 3};  // v is std::initializer_list<int>, not std::vector<int>
```

### 7.13 Lambda Expressions

- Capture by `[&]` or `[=]` only when the lambda is short and the captures are obvious
- For longer lambdas or lambdas in public APIs, capture individual variables explicitly
- Prefer `[&]` in local scope (cheaper than copying)
- Prefer `[=]` when the lambda outlives the current scope

```cpp
// Simple lambda — default capture is fine
std::sort(v.begin(), v.end(), [](int a, int b) { return a < b; });

// Explicit captures for clarity
std::thread t([this, &result, key]() {
  result = cache_.Lookup(key);
});

// Default capture in local scope
auto worker = [&]() {
  Process(data_, config_);
};
```

**Never use default capture by reference `[&]` in a lambda that outlives the current scope** — dangling references.

### 7.14 `std::move` and `std::forward`

- Use `std::move` to indicate ownership transfer
- Use `std::forward` for perfect forwarding (template forwarding references)
- Never `std::move` a `const` object — it decays to `const T&&` which can only bind to `const T&`

```cpp
// Good — move a non-const object
void Foo(std::string value);
std::string s = "hello";
Foo(std::move(s));  // s is now in a valid but unspecified state

// Bad — pointless, const prevents moving
const std::string s = "hello";
Foo(std::move(s));  // Actually copies (const T&& decays to const T&)

// Perfect forwarding
template <typename T>
void Wrapper(T&& arg) {
  Inner(std::forward<T>(arg));
}
```

Prefer passing by value and moving into place for sink parameters:

```cpp
class MyClass {
 public:
  // Good — pass by value, move into member
  void SetName(std::string name) { name_ = std::move(name); }

 private:
  std::string name_;
};
```

### 7.15 `auto*` vs `auto`

When deducing a pointer type, use `auto*` for clarity:

```cpp
// Good
auto* ptr = dynamic_cast<Base*>(derived);

// Confusing — could be a pointer, smart pointer, or iterator
auto ptr = dynamic_cast<Base*>(derived);
```

Use `const auto*` when the pointee is const:

```cpp
const auto* node = GetNode();  // node is const Node*
```

### 7.16 `noexcept`

Use `noexcept` to:
- Enable compiler optimizations (e.g., move operations on `std::vector`)
- Document that a function never throws

```cpp
// Move operations should be noexcept for optimal std::vector performance
MyClass(MyClass&& other) noexcept : data_(std::move(other.data_)) {}
MyClass& operator=(MyClass&& other) noexcept {
  data_ = std::move(other.data_);
  return *this;
}

// Swap should be noexcept
void swap(MyClass& a, MyClass& b) noexcept {
  using std::swap;
  swap(a.data_, b.data_);
}
```

Do not add `noexcept` speculatively — the function must truly never throw.

### 7.17 Undefined Behavior Avoidance

Undefined behavior is never acceptable in production code. Common pitfalls:

```cpp
// Signed integer overflow — UB
int x = INT_MAX;
x += 1;  // UB

// Dereferencing nullptr — UB
int* p = nullptr;
*p = 42;  // UB

// Use-after-free — UB
int* p = new int(42);
delete p;
*p = 0;  // UB

// Out-of-bounds access — UB
int arr[5];
arr[5] = 42;  // UB

// Uninitialized variable — UB
int x;
Use(x);  // UB
```

### 7.18 Threading

- Use standard threading primitives (`<thread>`, `<mutex>`, `<condition_variable>`)
- Prefer `std::lock_guard` or `std::unique_lock` over manual `lock()`/`unlock()`
- Use `std::atomic` for simple atomic operations (counters, flags)
- Avoid raw platform threading APIs (pthreads, Win32 threads) unless absolutely necessary

```cpp
#include <mutex>
#include <thread>

class Counter {
 public:
  void Increment() {
    std::lock_guard<std::mutex> lock(mutex_);
    ++count_;
  }

  int GetCount() const {
    std::lock_guard<std::mutex> lock(mutex_);
    return count_;
  }

 private:
  mutable std::mutex mutex_;
  int count_ = 0;
};
```

---

## 8. Review Checklist

Before submitting C++ code for review, verify:

- [ ] All headers are self-contained (compile alone)
- [ ] #define guards follow `PROJECT_PATH_FILE_H_` format
- [ ] Includes follow the IWYU principle and the correct group order (related → C → C++ → libs → project)
- [ ] No forward declarations of non-owned entities where `#include` could work
- [ ] No `using namespace` directives
- [ ] Class data members end with `_`; struct data members do not
- [ ] Single-argument constructors are `explicit`
- [ ] Virtual overrides use `override`
- [ ] No C-style casts
- [ ] No RTTI (`typeid`/`dynamic_cast`) in production code
- [ ] No variable-length arrays (VLA)
- [ ] No undefined behavior (signed overflow, out-of-bounds, use-after-free)
- [ ] Line length ≤ 80 characters
- [ ] Indentation is 2 spaces, no tabs
- [ ] K&R brace style followed
- [ ] Comments use `//` style, are proper sentences with punctuation
- [ ] `const` used wherever values are immutable
- [ ] `constexpr` used for compile-time constants
- [ ] Fixed-width integers (`int32_t`, etc.) preferred over `int`/`long`
- [ ] `auto` used appropriately (not in public API return types)
- [ ] No macros when `constexpr`, inline, template, or enum alternatives exist
- [ ] `std::move` not used on `const` objects
- [ ] Move constructors/assignment marked `noexcept` where possible
- [ ] Lambdas have explicit captures (not default capture) when non-trivial
- [ ] Thread-safe code uses `std::mutex`/`std::atomic` correctly
- [ ] `TODO` items have associated bug/owner references

---

Full source: `G:\styleguide\cppguide.html`
