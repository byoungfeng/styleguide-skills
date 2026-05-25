# Google Python Style Guide — Deep Reference

Use this for thorough code review. It covers edge cases, detailed examples, and disallowed patterns. For day-to-day coding, start with `SKILL.md`. Full source: `G:\styleguide\pyguide.md` (3,709 lines).

---

## 1. Lint (2.1)

Run `pylint` on all code. Suppress false positives inline:

```python
def do_PUT(self):  # WSGI name, so pylint: disable=invalid-name
```

- Use `pylint: disable` (not the deprecated `pylint: disable-msg`).
- Unused arguments: delete the variable at function start with a comment:
  ```python
  def viking_cafe_order(spam: str, beans: str, eggs: str | None = None) -> str:
      del beans, eggs  # Unused by vikings.
      return spam + spam + spam
  ```
- Using `_` as unused prefix or assigning to `_` is allowed but discouraged — it breaks callers passing by name.

---

## 2. Imports (2.2 + 3.13)

### Approved patterns
```python
# Modules/packages only — never individual types
import x
from x import y         # y is a module, not a class/function
from x import y as z    # Only when: name collision, long name, too generic
import numpy as np      # Standard abbreviations only
```

### Disallowed
```python
from x import SomeClass     # Don't import individual classes
import os, sys              # Never comma-chain imports
from . import sibling       # No relative imports — use full path
```

### Ordering
1. Standard library imports
2. Third-party imports
3. Local module imports

Each group separated by a blank line. Alphabetical within groups is nice but not required.

### Typing imports exemption
Symbols from `typing`, `collections.abc`, and `typing_extensions` are exempt — import them directly.

---

## 3. Packages (2.3)

Always use the full package path. Never relative imports.

```python
# Yes
import absl.flags
from absl import flags

# No — relative
from . import sibling
```

---

## 4. Exceptions (2.4)

- Derive exceptions from `Exception`, not `BaseException`.
- Use specific built-in exceptions (`ValueError`, `TypeError`) when appropriate.
- Never use `except:` (catch-all). Always specify the exception type.
- Don't use exceptions for control flow. Exceptions are for error conditions.
- `try` blocks should minimize the code inside them.
- Use `raise` without arguments to re-raise the current exception in an `except` block.
- Use `raise from` for exception chaining.
- Use `assert` only to check internal invariants (optimized away with `-O`).

---

## 5. Mutable Global State (2.5)

Avoid. If unavoidable:
- Make it module-level
- Use `_` prefix for internal state
- At import time, state must not change

```python
_counter: int = 0  # Module-level variable, acceptable if scoped
```

---

## 6. Comprehensions & Generator Expressions (2.7)

**OK for simple cases.** Limit: no more than 2 `for` clauses or 2 filter conditions + one loop level.

```python
# Good
result = [x for x in range(10) if x % 2 == 0]
result = {x: x**2 for x in range(10)}

# Bad — too complex, use a for loop instead
result = [(x, y) for x in range(10) for y in range(5) if x != y if x > y]
```

Generator expressions are preferred to list comprehensions when the result feeds directly into another function:
```python
sum(x**2 for x in range(10))  # No intermediate list
```

---

## 7. Default Iterators and Operators (2.8)

Use built-in iteration protocols. Don't call `.keys()`, `.items()` is fine.

```python
# Yes
for key in dct: ...
if key in dct: ...

# No
for key in dct.keys(): ...
if dct.has_key(key): ...    # Python 2 relic
```

---

## 8. Lambda Functions (2.10)

OK for single expressions. If longer than ~60 chars (including args, colon, expression), write a named function:

```python
# Good
sorted(users, key=lambda u: u.last_login)

# Bad — too long
sorted(users, key=lambda u: u.last_login.timestamp() if u.last_login else 0)
```

---

## 9. Conditional Expressions (2.11)

OK for simple value selection. Not for complex logic:

```python
# Good
x = a if condition else b

# Bad
x = (complex_call(1) and another_call(2)) if condition else (fallback(3) if other else default(4))
```

---

## 10. Default Argument Values (2.12)

**Never use mutable objects as defaults:**

```python
# Bad
def f(x, items=[]): ...

# Good
def f(x, items=None):
    if items is None:
        items = []
```

Applies to: `[]`, `{}`, `set()`, and any mutable object.

---

## 11. Properties (2.13)

Use `@property` for simple attribute access. Keep them cheap — no I/O, no complex computation.

Use `@property` plus a setter instead of `get_foo()` / `set_foo()` methods:

```python
class Rectangle:
    def __init__(self, width, height):
        self._width = width
        self._height = height

    @property
    def area(self):
        return self._width * self._height
```

Only use when there is an actual computation or validation. For plain data, use regular attributes.

---

## 12. True/False Evaluations (2.14)

Use the implicit falseness/truthfulness of values:

```python
# Good
if not users: ...
if items: ...

# Bad
if len(users) == 0: ...
if len(items) > 0: ...

# Be explicit with None
if x is not None: ...  # Good
if not x is None: ...  # Bad
```

Be aware of edge cases: `0`, `""`, and empty containers are falsy. If you specifically need `None`, use `is None` / `is not None`.

Never compare a boolean with `==`:
```python
# Bad
if greeting == True: ...
if greeting is True: ...  # Worse
```

---

## 13. Lexical Scoping (2.16)

Python's scoping rules are fine. Don't use `global` or `nonlocal` unless absolutely necessary.

---

## 14. Function and Method Decorators (2.17)

- Must use `@functools.wraps` on all decorators that wrap functions.
- Use decorators judiciously — prefer explicit over implicit.
- Decorators that mutate signatures (add/remove/change parameters) are strongly discouraged.

---

## 15. Threading (2.18)

Don't use threading. If you must, prefer `concurrent.futures` with `ThreadPoolExecutor` or async/await with `asyncio`.

---

## 16. Power Features (2.19)

Avoid metaclasses, `__new__`, descriptor protocol, `__del__`, contextvars, monkey-patching — unless the problem cannot be solved without them.

---

## 17. `__future__` Imports (2.20)

Use `from __future__ import annotations` when you need forward references in type hints. Place it as the first import.

---

## 18. Type Annotations — Deep Dive (2.21 + 3.19)

### Must annotate
- All function/method signatures (parameters and return type)
- Module-level and class-level variables when inference isn't obvious

### Union syntax
Prefer `X | Y` over `Union[X, Y]`, `X | None` over `Optional[X]`:

```python
def compute(values: list[int], factor: float | None = None) -> dict[str, float]:
    ...
```

### Variables
Annotation on same line before assignment:
```python
items: list[str] = []
config: dict[str, int] | None = None
```

### Tuples vs Lists
Use `list[T]` for homogeneous sequences of variable length.  
Use `tuple[T, ...]` for homogeneous fixed-length.  
Use `tuple[T1, T2]` for heterogeneous (e.g., `tuple[str, int]` for name+age).

### Forward declarations
Use `from __future__ import annotations` (PEP 563) for forward references:
```python
from __future__ import annotations

class Node:
    def get_children(self) -> list[Node]:  # Node not yet fully defined
        ...
```

### Type aliases
```python
Vector = list[float]
UserId = int
```

### String types
Use `str` for all text. Avoid `typing.Text` (Python 2 compat).

### Conditional typing imports
```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from other_module import HeavyType
```

### Generics
Use built-in generics: `list[T]`, `dict[K, V]`, `tuple[T, ...]`.
For custom generic types, use `TypeVar`:

```python
from typing import TypeVar
_T = TypeVar("_T")

class Stack(list[_T]):
    def pop(self) -> _T: ...
```

---

## 19. Line Length — Edge Cases (3.2)

The 80-char limit is relaxed ONLY when Black/Pyink won't fix it. In all other cases, break the line.

Never use backslash `\` for line continuation. Use parentheses:

```python
# Yes
very_long_variable = (
    "this is a very long string that "
    "needs to wrap to the next line"
)

# No
very_long_variable = "this is a very long string that " \
                     "needs to wrap to the next line"
```

---

## 20. Indentation — Details (3.4)

Always 4 spaces.

For wrapped constructs, choose a consistent style:
1. Align with opening delimiter
2. Hanging 4-space indent

```python
# Aligned
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# Hanging indent (4 spaces)
foo = long_function_name(
    var_one, var_two,
    var_three, var_four,
)
```

Trailing commas: use them in multi-line sequences. It reduces diff noise when adding items.

Closing brackets: can go at the end of the last line, or on their own line indented to match the opening line.

---

## 21. Whitespace — Details (3.6)

The PEP 8 rules in summary:

**Spaces around:**
- Assignment: `x = 1`
- Operators: `x + y`, `x > y`
- After comma: `f(1, 2, key=value)`
- After colon in dict: `{'key': value}`

**No spaces around:**
- `=` in keyword arguments: `f(key=value)` ✓, `f(key = value)` ✗
- Inside brackets/parens: `[1]` ✓, `[ 1 ]` ✗
- Slices except when colon is needed: `x[0:5]`

**No trailing whitespace anywhere.**

---

## 22. Shebang (3.7)

Use `#!/usr/bin/env python3` for executable scripts. Most modules don't need it.

---

## 23. Docstrings — Detailed Format (3.8)

### Module docstrings
At the top of the file. Describe contents and usage.
```python
"""A module for processing spam orders and generating invoices."""
```

Test modules are exempt from needing a docstring.

### Function/method docstrings
One-line summary first, then details. Covers: what it does, args, returns, raises.
```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table, using other_silly_variable for flavor.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each row to fetch.
        other_silly_variable: Another optional variable, for fun.

    Returns:
        A dict mapping keys to the corresponding table row data.
        Each row is represented as a tuple of strings.

    Raises:
        IOError: An error occurred accessing the bigtable.
    """
```

### Class docstrings
After the class definition line. Describe the class and its public attributes.

### Block/inline comments
- Use `# ` (hash + space) for inline comments.
- Block comments above the code they describe, indented to match.

---

## 24. Strings — Details (3.10)

- Prefer **f-strings** for interpolation.
- Use `repr()` for human-readable, `str()` for machine-parseable.
- For logging: use `%` formatting, and pass values as args to the logging function (lazy evaluation).
  ```python
  logging.info("Processing %s at %s", item, timestamp)  # Good
  logging.info(f"Processing {item} at {timestamp}")     # Works but pre-evaluates
  ```

---

## 25. File/Socket Resources (3.11)

Use `with` (context manager). Only use explicit `.close()` when the lifetime spans beyond a single `with` block.

```python
# Good
with open(path) as f:
    data = f.read()

# Acceptable — lifetime beyond with
f = open(path, 'rb')
reader = SomeReader(f)
reader.read()
f.close()  # Explicit because reader holds the handle
```

---

## 26. TODO Comments (3.12)

Format: `# TODO(username): description`

```python
# TODO(alice): Implement rate limiting for this endpoint.
# TODO(bob): Remove this workaround after bug #1234 is fixed on the server side.
```

---

## 27. Statements (3.14)

One statement per line. Exception: `if` with trivial body on same line is OK.

```python
# OK
if x > 0: return x

# Better
if x > 0:
    return x
```

No nested `if`/`for`/`while` on single line: `for x in items: if x > 0: yield x` — break it up.

---

## 28. Getters and Setters (3.15)

Use `@property` for simple attribute access. Use `@prop.setter` when mutation needs validation.

When the getter/setter does I/O, complex computation, or throws, use `get_foo()` / `set_foo()` methods instead — so the caller knows they're expensive or fallible.

---

## 29. Naming — Deep Dive (3.16)

### Anti-patterns
- Single-character names beyond `i`, `j`, `e`, `f`
- Dashes in any package/module name
- `__double_underscore__` (Python reserved)
- Hungarian notation or type-embedded names (`str_name`, `list_items`)
- One-letter differences between names in the same scope

### Guidelines
- Descriptiveness should be proportional to the scope of visibility. An `i` is fine in a 5-line block, bad across nested scopes.
- Mathematical code may use short names matching established notation (cite the source, narrowly disable pylint).

---

## 30. Main (3.17)

Every file that runs as a script must use:
```python
def main():
    ...

if __name__ == '__main__':
    main()
```

---

## 31. Function Length (3.18)

No hard limit, but prefer small and focused. If > 40 lines, consider whether decomposition improves readability.

---

## Review Checklist

Quick checklist for code review:

- [ ] `pylint` runs clean (or suppressions are justified)
- [ ] Imports use full path, ordered stdlib → third-party → local
- [ ] No mutable default arguments
- [ ] No bare `except:`
- [ ] No backslash line continuation
- [ ] 4-space indentation, no tabs, ≤80 chars (within reason)
- [ ] Type annotations on all function signatures
- [ ] Docstrings on all public modules/functions/classes
- [ ] Naming follows the convention table
- [ ] Properties used instead of getters/setters
- [ ] Context managers used for file/socket resources
- [ ] `if __name__ == '__main__'` for executables
- [ ] TODO comments have username: `# TODO(name): ...`

---

Source: `G:\styleguide\pyguide.md` (Google Python Style Guide, CC-By 3.0)
