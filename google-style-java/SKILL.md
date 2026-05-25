---
name: google-style-java
description: Google Java Style Guide — naming, formatting, Javadoc, imports, and language conventions. Use when writing, reviewing, or refactoring Java code to apply Google's coding standards. Triggers on any Java coding task.
---

# Google Java Style Guide — Quick Reference

## Naming Conventions

| Identifier | Convention | Example |
|---|---|---|
| Packages | all lowercase, no underscores | `com.example.myapp` |
| Classes | UpperCamelCase | `XmlHttpRequest`, `CustomerDao` |
| Methods | lowerCamelCase | `getUserName()`, `sendResponse()` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT` |
| Non-constant fields | lowerCamelCase | `customerName`, `totalCount` |
| Parameters | lowerCamelCase | `inputStream`, `errorMessage` |
| Local variables | lowerCamelCase | `itemList`, `currentValue` |
| Type variables | single capital letter or NameT | `E`, `K`, `V`, `RequestT` |
| Unnamed variables | `_` (Java 21+) | `try { } catch (_) { }` |

## Formatting

- **Column limit**: 100 characters
- **Indent**: 2 spaces, no tabs
- **Braces**: K&R style (line end, not next line)
- **Braces always used**: even for single-line bodies
- **Line wrapping**: break at higher syntactic level
- **Continuation indent**: +4 spaces

### K&R Brace Style

```java
class MyClass {
  void myMethod() {
    if (condition) {
      doSomething();
    } else {
      doOther();
    }
  }
}
```

### Line Wrapping Example

```java
// Break at higher syntactic level, continued lines indented +4
return someMethod(longExpression1, longExpression2,
    longExpression3, longExpression4);
```

## Source File Structure

1. License (if any)
2. Package statement
3. Import statements (no wildcards, ASCII order)
4. Exactly one top-level class

```java
package com.example.myapp;

import com.example.common.Thing;
import com.example.common.Util;

import java.util.List;
import java.util.Map;

public class MyClass {
  // ...
}
```

## Imports

- **No wildcard imports**: `import java.util.*;` is forbidden
- **ASCII sort order**: `com.example.a` before `com.example.b`, `java.util.List` before `javax.sql.DataSource`
- **No line-wrapping** in import statements
- **No static import for classes** — only static methods/fields

## Javadoc

```java
/**
 * A brief summary fragment (period-terminated sentence).
 *
 * <p>Additional details in HTML paragraphs.
 *
 * @param paramName description of parameter
 * @return description of return value
 * @throws IOException if an I/O error occurs
 */
public Result process(String paramName) throws IOException {
  // ...
}
```

- **Required**: all public/protected classes, methods, and constants
- **Not required**: self-explanatory simple methods like getters/setters
- Use `/** ... */` format (not `/* ... */` or `//`)

## Language Rules

| Rule | Detail |
|---|---|
| No wildcard imports | `import java.util.*;` prohibited |
| No C-style arrays | `String[] args` not `String args[]` |
| `@Override` | Always used when overriding superclass/interface method |
| `@Nullable` / `@NonNull` | Annotate nullable/non-null parameters and returns |
| Immutable collections | Prefer `ImmutableList`, `ImmutableMap`, etc. |
| Auto-value | Prefer for simple value types |

## Camel Case (acronyms as whole words)

| Correct | Incorrect |
|---|---|
| `XmlHttpRequest` | `XMLHTTPRequest` |
| `parseDnsRecord` | `parseDNSRecord` |
| `supportsSsl` | `supportsSSL` |
| `HttpConnection` | `HTTPConnection` |

Acronyms like `HTTP`, `XML`, `DNS` are treated as whole words when camel-casing: only first letter is capitalized in UpperCamelCase, only first letter of non-first words is capitalized in lowerCamelCase.

---

Full source: `G:\styleguide\javaguide.html`
