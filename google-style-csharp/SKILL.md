---
name: google-style-csharp
description: Google C# Style Guide — naming, formatting, organization, and coding conventions. Use when writing, reviewing, or refactoring C# code to apply Google's coding standards. Triggers on any C# coding task.
---

# Google C# Style Guide — Quick Reference

## Quick Reference

### Naming

| Category | Convention | Examples |
|---|---|---|
| Classes, Methods, Enums, Public fields/properties, Namespaces | PascalCase | `class CustomerService` |
| Local variables, Parameters | camelCase | `int itemCount` |
| Private/protected/internal fields and properties | `_camelCase` | `_connectionString` |
| Interfaces | Prefix `I` + PascalCase | `ICustomerRepository` |
| Acronyms | PascalCase as whole words | `MyRpc` not `MyRPC` |
| Filenames | PascalCase, match main class name | `CustomerService.cs` |

### Organization

- **Modifier order**: `public protected internal private new abstract virtual override sealed static readonly extern unsafe volatile async`
- **`using` directives**: at top, before namespace; System imports first, then alphabetical
- **Class member order**: nested types → static/const/readonly fields → fields/properties → constructors/finalizers → methods
- **Member visibility order within each group**: public → internal → protected internal → protected → private

### Formatting

- 2-space indent, no tabs
- Column limit 100
- One statement per line
- K&R braces (Egyptian braces) always used
- Space after `if`/`for`/`while`/`catch`/`switch`, no space inside parentheses
- Braces on same line as statement
- Line continuation: +4 indent

### Coding Guidelines

- Prefer `const` then `readonly` over magic values
- Use `var` when type is obvious, avoid when it obscures readability
- Prefer `List<T>` over arrays for public APIs
- Prefer LINQ member syntax (method chain) over SQL-style query syntax
- Use expression body for simple properties and methods
- Prefer named classes over `Tuple<T>` / `ValueTuple`
- Use string interpolation over concatenation or `string.Format()`

Full source: `G:\styleguide\csharp-style.md`
