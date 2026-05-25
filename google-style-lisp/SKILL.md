---
name: google-style-lisp
description: Google Common Lisp Style Guide — naming, formatting, documentation, packages, and language usage. Use when writing, reviewing, or refactoring Common Lisp code to apply Google's coding standards. Triggers on any Common Lisp coding task.
---

# Google Common Lisp Style Guide — Quick Reference

## Formatting

- **Line length**: 100 characters maximum.
- **Indentation**: Indent like a properly configured GNU Emacs (use `cl-indent`). No tabs.
- **Parentheses**: No extra spaces around parentheses. No lonely parentheses on their own line.
- **Vertical whitespace**: One blank line between top-level forms.
- **Alignment**: When breaking across lines, align nested forms vertically.

## Documentation

- **Docstrings**: Required on all functions, classes, variables, and macros.
- **Comment semicolons**:
  - `;;;;` — file headers
  - `;;;` — groups of top-level forms
- `;;` — inside a form (body comments)
  - `;` — end-of-line comments
- **TODO**: Use `TODO(name): comment`.

## Naming

- **Case**: lowercase.
- **Word separator**: hyphens (`-`), not slashes or dots.
- **Global constants**: `+earmuffs+`.
- **Globals**: `*earmuffs*`.
- **Predicates**: end in `P` or `-P`.
- **Library prefix**: none in symbol names.

## Packages

- Use appropriately. Don't shadow CL symbols. Don't use `::` double-colons in production code. Prefer not to `:use` other packages.

## Language

- Prefer mostly functional style — avoid side effects. Favor iteration over recursion. Use `loop` or `iter`.

## Full source

`G:\styleguide\lispguide.xml`
