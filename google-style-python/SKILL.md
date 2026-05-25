---
name: google-style-python
description: Google Python Style Guide — naming, formatting, imports, type annotations, and best practices. Use when writing, reviewing, or refactoring Python code to apply Google's coding standards. Triggers on any Python coding task.
---

# Google Python Style Guide (Summary)

Use this as your day-to-day reference for writing Python code that follows Google's conventions. For deeper review, read `references/deep.md`. The full canonical guide is at `G:\styleguide\pyguide.md`.

## Naming Conventions

| Type | Public | Internal |
|---|---|---|
| Packages | `lower_with_under` | |
| Modules | `lower_with_under` | `_lower_with_under` |
| Classes | `CapWords` | `_CapWords` |
| Exceptions | `CapWords` | |
| Functions | `lower_with_under()` | `_lower_with_under()` |
| Global/Class Constants | `CAPS_WITH_UNDER` | `_CAPS_WITH_UNDER` |
| Global/Class Variables | `lower_with_under` | `_lower_with_under` |
| Instance Variables | `lower_with_under` | `_lower_with_under` (protected) |
| Method Names | `lower_with_under()` | `_lower_with_under()` (protected) |
| Function/Method Parameters | `lower_with_under` | |
| Local Variables | `lower_with_under` | |

### Key naming rules
- **Be descriptive.** Avoid abbreviations, especially ambiguous ones or those that delete letters within words.
- **No dashes** in package/module names. No `__double_underscore__` (Python reserved).
- **Don't embed type names** in variable names (`id_to_name_dict` → use `id_to_name`).
- **Single-char names** only for counters (`i`, `j`), exception handler (`e`), file handle (`f`), or established math notation.
- **Internal (protected)** members use a single leading underscore. Avoid double-underscore name mangling — it's not truly private.
- **Test files** follow PEP 8: `test_<method_under_test>_<state>`. Dashes are not allowed in filenames.

## Formatting

### Line Length
- **80 characters** maximum.
- Exceptions: long imports, URLs/paths in comments, long string module-level constants, pylint disable comments.
- Use **implicit line joining** (parentheses, brackets, braces) — never use backslash `\` for continuation.
- Break at the **highest possible syntactic level**.

### Indentation
- **4 spaces.** Never tabs.
- Wrapped lines: align with opening delimiter, or use hanging 4-space indent.

### Blank Lines
- Two lines between top-level definitions.
- One line between method definitions.

### Whitespace
- Follow PEP 8: no trailing whitespace, spaces around operators, after commas, around `=` for keyword args only.
- No whitespace immediately inside brackets/parens: `spam(ham[1], {eggs: 2})`.

### Semicolons
- Don't use them. One statement per line.

### Parentheses
- Use sparingly. Don't wrap `if`/`return` conditions unless implicitly continuing a line.

## Imports

### Import style
- **Use `import x`** for packages and modules. Don't import individual types/functions.
- **Use `from x import y`** where `x` is the package prefix and `y` is the module name.
- **Use `from x import y as z`** when: two modules named `y` conflict, `y` conflicts with a name in scope, or `y` is inconveniently long.
- **Use `import y as z`** only for standard abbreviations (e.g., `import numpy as np`).

### Import ordering
- Standard library imports first, then third-party, then local modules.
- One import per line. No `import os, sys`.
- Full package path — no relative imports.
- Typing imports (`typing`, `collections.abc`, `typing_extensions`) follow the `from` pattern.

## Type Annotations

- **Must be used** for all new Python code (per PEP 484/526).
- Use `|` union syntax (e.g., `str | None`) instead of `Optional[str]`.
- Variables are declared on their own line, with the annotation, before assignment:
  ```python
  code: int = 0
  ```
- Use `from __future__ import annotations` for forward references when needed.
- Import types with `from typing import ...` for types that aren't built-in (e.g., `TypeVar`, `Protocol`).

## Language Rules (Quick Reference)

| Rule | Do |
|---|---|
| **Lint** | Run `pylint`. Suppress false positives with `# pylint: disable=name`. |
| **Exceptions** | Use for error conditions. Never for control flow. Derive from `Exception`. |
| **Mutable globals** | Avoid. Either pass state explicitly or use an immutable singleton. |
| **Default args** | Never use mutable defaults (`def f(x=[])`). Use `None` + check. |
| **Comprehensions** | OK for simple cases. No more than 2 `for` clauses or filter conditions. |
| **Lambda** | OK for one-liners. If longer than ~60 chars, write a named function. |
| **Conditional expressions** | OK for simple cases: `x = a if condition else b`. |
| **Properties** | Use `@property` instead of getters/setters. Keep them cheap. |
| **Truthiness** | Use implicit boolean checks: `if items:` not `if len(items) > 0:`. |
| **Decorators** | Judiciously. Must have `functools.wraps`. |
| **Threading** | Avoid. If needed, prefer `concurrent.futures`. |
| **Power features** | Avoid metaclasses, custom `__new__`, descriptor protocols unless strictly necessary. |
| **`__future__`** | Use `from __future__ import annotations` when needed for forward refs. |

## Strings

- Use **f-strings** or `%` for interpolation. Avoid `.format()`.
- Use `repr()` for output meant for humans; use `str()` for machine parseable output.
- Prefer `.join()` over `+` for concatenating many strings.
- Avoid `+` or `+=` in a loop — use `"".join(list)` instead.

## Comments & Docstrings

- **Modules**: docstring describing contents and usage. Test modules don't need one.
- **Functions/methods**: docstring describing what it does, args, returns, raises. One-line summary line first.
- **Classes**: docstring under the class definition.
- **TODO**: format as `# TODO(name): description` — includes your username.
- Docstrings use triple-double-quotes `"""`.

## Files, Sockets, and Stateful Resources

- Use `with` statement (context manager) to ensure cleanup.
- Explicitly call `.close()` only if the lifetime extends beyond the `with`.

## Main

```python
def main():
    ...

if __name__ == '__main__':
    main()
```

## Function Length

- Be reasonable. If a function exceeds ~40 lines, consider if it should be split.

## Tooling

- Use **Black** or **Pyink** auto-formatter to avoid formatting debates.
- Use **pylint** with Google's pylintrc: `https://google.github.io/styleguide/pylintrc`

## Full Reference

- Canonical source: `G:\styleguide\pyguide.md`
- Deep reference (code review): `references/deep.md`
