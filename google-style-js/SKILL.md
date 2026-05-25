---
name: google-style-js
description: Google JavaScript Style Guide — naming, formatting, JSDoc, modules, and language conventions. Use when writing, reviewing, or refactoring JavaScript code to apply Google's coding standards. Triggers on any JavaScript coding task.
---

# Google JavaScript Style Guide — Quick Reference

> **Deprecation notice:** This guide is no longer being updated. Google recommends migrating to TypeScript.

## Naming conventions

| Category | Convention | Example |
|---|---|---|
| Packages | `lowerCamelCase` | `google.engine` |
| Classes / interfaces / types | `UpperCamelCase` | `class Request {}` |
| Methods | `lowerCamelCase` (private: optional trailing `_`) | `sendRequest()` / `sendRequest_()` |
| Constants | `CONSTANT_CASE` | `MAX_RETRY_COUNT` |
| Non-constant fields | `lowerCamelCase` (private: optional trailing `_`) | `this.responseCount_` |
| Parameters | `lowerCamelCase` | `function get(url, options)` |
| Local variables | `lowerCamelCase` | `let total = 0;` |
| Template params | `ALL_CAPS` | `function map(iterable, func) /** @template T */` |

## Formatting basics

- 80-character column limit
- 2-space indent, no tabs
- K&R brace style (1TBS)
- One statement per line
- Spaces inside non-empty block braces (`{ }`)
- No spaces inside parentheses

## Source file structure

Use `goog.module` or ES modules. Order: license header → `@fileoverview` JSDoc → `goog.module` / ES imports → implementation. Named exports only — default exports are banned.

```js
goog.module('my.app.Car');

const Engine = goog.require('my.app.Engine');

class Car {
  constructor() {
    this.engine_ = new Engine();
  }
}
exports = Car;
```

## JSDoc quick reference

| Tag | Usage |
|---|---|
| `/** @param {type} name description */` | Document parameter |
| `/** @return {type} description */` | Document return value |
| `@private` | Visibility |
| `@const` | Constant marker |
| `@final` | Prevent override |
| `@export` | Export for compiler |
| `@template T` | Generic type param |
| `@implements {Interface}` | Implement interface |
| `@override` | Override method |
| `@fileoverview` | File-level comment |
| `@type {type}` | Inline type |
| `@throws {type}` | Document exception |

## Language rules

- `const` / `let` only — no `var`
- No Array constructor with non-single arguments
- No `with` / `eval`
- No sparse arrays
- Prefer standard features over non-standard
- Use braces for all multi-line blocks
- `===` / `!==` always, never `==` / `!=`

## Camel case definition

Start with a letter, then any mix of letters, digits, and underscores. Upper camel: first letter capital. Lower camel: first letter lowercase. Acronyms and initialisms are treated as whole words (e.g., `loadHttpUrl`, not `loadHTTPUrl`).

---

Full source: `G:\styleguide\jsguide.html`
