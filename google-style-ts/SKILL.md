---
name: google-style-ts
description: Google TypeScript Style Guide — naming, formatting, type system, imports/exports, and language conventions. Use when writing, reviewing, or refactoring TypeScript code to apply Google's coding standards. Triggers on any TypeScript coding task.
---

# Google TypeScript Style Guide — Quick Reference

## Naming Conventions

| Category | Style | Examples |
|---|---|---|
| Class / Interface / Type / Enum / Decorator / Type parameters | `UpperCamelCase` | `class UserService`, `interface UserData`, `type JsonMap`, `enum Color`, `@Controller`, `<T>` |
| Variable / Parameter / Function / Method / Property / Module alias | `lowerCamelCase` | `const count = 1`, `function getUser()`, `method send()`, `property: string`, `import * as modAlias` |
| Global constants / Enum values | `CONSTANT_CASE` | `const MAX_SIZE = 100;`, `enum Color { RED = 1 }` |

**Rules:**
- No `_` prefix or suffix (even for private members — use TypeScript `private`)
- No `I` prefix for interfaces (`interface User` not `interface IUser`)
- No `_` unused parameter prefix — use `_` as the full name if truly unused
- Abbreviations are treated as words: `loadHttpUrl`, `parseXml`, `newUrl`

## Formatting

- **Column limit:** 80 characters
- **Indentation:** 2 spaces, no tabs
- **Braces:** K&R style (opening brace on same line)
- **Semicolons:** Required
- **Declarations:** `const` by default, `let` when reassignment needed, **no `var`**
- **One variable per declaration:** `const a = 1; const b = 2;` not `const a = 1, b = 2;`

## Imports

- ES6 module syntax (`import`/`export`) throughout
- Named exports preferred over default exports
- `import type` for type-only imports: `import type { Foo } from './foo'`
- No `require()` calls, no `namespace`, no `<reference>` directives
- Use relative imports for project code, absolute for external packages
- No circular imports

## Type System

- Prefer type inference over explicit type annotations
- Use `T | U` union syntax (no `T | null | undefined`)
- Use optional `?` over `| undefined`: `foo?: string` not `foo: string | undefined`
- Use structural types (duck typing) — avoid nominal typing patterns
- Avoid `any` — use `unknown` when the type is truly unknown
- Use `readonly` for immutability
- Prefer `interface` over `type` for object shapes
- Use `as` for type assertions (not angle-bracket syntax)

## Language Features

- `const` / `let` only — no `var`
- No `Array()` constructor — use `[]` or `Array.from()`
- No `for...in` — use `for...of` or `.forEach()`
- Arrow functions preferred for callbacks (preserve lexical `this`)
- Template strings over string concatenation
- Use `===` / `!==` (not `==` / `!=`)
- Use `for...of` with `.entries()` when index is needed

---

Full source: `G:\styleguide\tsguide.html`
