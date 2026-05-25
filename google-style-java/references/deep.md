# Google Java Style Guide — Deep Reference

---

## 1. Source File Basics

### 1.1 File Name

The source file name must be the case-sensitive name of the top-level class it contains, plus the `.java` extension.

```java
// File: ResultParser.java
public class ResultParser {
  // ...
}
```

### 1.2 File Encoding

All source files must be encoded in **UTF-8**.

### 1.3 Special Characters

#### Whitespace Characters

Use only the **ASCII space** character (0x20) for indentation. The tab character (0x09) is prohibited.

#### Special Escape Sequences

Any character with a special escape sequence (`\b`, `\t`, `\n`, `\f`, `\r`, `\"`, `\\`) must be represented using the escape sequence, not the actual character.

```java
// Correct
String value = "line1\nline2";

// Incorrect — literal newline in source
String value = "line1
line2";
```

#### Non-ASCII Characters

Non-ASCII characters may appear as Unicode escape sequences or as the actual character. Prefer the actual character where it makes the code more readable.

```java
// Both acceptable — preference for actual character
String unit = "µs";         // actual character
String unit = "\u00b5s";   // Unicode escape
```

---

## 2. Source File Structure

A source file consists of, **in order**:

1. License or copyright information (if present)
2. Package statement
3. Import statements
4. Exactly one top-level class

A blank line separates each section that is present.

### 2.1 License

If a license or copyright header belongs in the file, it goes at the very top as a block comment.

```java
/*
 * Copyright 2026 Google LLC
 *
 * Licensed under the Apache License, Version 2.0 ...
 */
```

### 2.2 Package Statement

The package statement is **not line-wrapped** (the column limit does not apply).

```java
package com.example.deeply.nested.package.name;
```

### 2.3 Import Statements

#### No Wildcard

Wildcard imports — static or otherwise — are **not permitted**.

```java
// Forbidden
import java.util.*;
import com.example.common.*;
```

#### No Line-Wrap

Import statements must not be line-wrapped (column limit does not apply).

```java
// Correct — long, but no wrapping
import com.example.some.really.long.package.name.SomeClassName;
```

#### Ordering

Imports are ordered in ASCII sort order. Within a group, imports are case-sensitive.

```java
import com.example.alpha.Something;         // before
import com.example.beta.Other;
import com.example.Beta.Other;               // 'B' sorts before 'a' in ASCII

import java.util.List;

import javax.sql.DataSource;
```

Two blank lines between non-static and static imports (if both present).

```java
import java.util.List;
import java.util.Map;

// blank line
// blank line
import org.junit.jupiter.api.Test;
```

#### No Static Import for Classes

Static import is not used for importing nested classes; use regular import instead.

```java
// Forbidden
import com.example.OuterClass.InnerClass;

// Correct — regular import of InnerClass
import com.example.InnerClass;
```

### 2.4 Class Declaration

#### Exactly One Top-Level Class

Each source file contains **exactly one** top-level class. A top-level class is one that is not enclosed in another class.

#### Member Ordering

Members of a class should be ordered in a logical, consistent manner. The suggested order:

1. Static fields (constants first, then other static fields)
2. Instance fields
3. Constructors
4. Methods (static methods grouped, instance methods grouped)
5. Inner classes/enums

#### Overloads

Methods that overload each other **must never be split** by other methods. They must appear sequentially with no other code in between.

```java
public void foo(int a) { }

public void foo(String b) { }

// public void bar() { }    ← would be incorrectly placed between overloads
```

---

## 3. Formatting Deep Dive

### 3.1 Braces

#### K&R Style (Nonempty Blocks)

Braces follow the Kernighan & Ritchie style ("Egyptian brackets"):

- No line break before the opening brace
- Line break after the opening brace
- Line break before the closing brace
- Line break after the closing brace (unless followed by `else`, `catch`, `finally`, or a continuation keyword)

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

#### Empty Blocks

An empty block or construct may appear as `{}` (concise). Exception: if it is part of a multi-block statement like `if/else` or `try/catch/finally`, it must have line breaks.

```java
// Acceptable
void doNothing() {}

// Required when part of multi-block
try {
  doSomething();
} catch (Exception e) {
  // empty — must have braces with line break
}
```

### 3.2 Block Indentation

Each time a new block or block-like construct is opened, the indent increases by **+2 spaces**.

```java
class Foo {
  void bar() {
    if (condition) {
      for (int i = 0; i < 10; i++) {
        nestedStuff();
      }
    }
  }
}
```

### 3.3 One Statement Per Line

Each statement must be followed by a line break.

```java
// Correct
int x = 1;
int y = 2;

// Forbidden
int x = 1; int y = 2;
```

### 3.4 Column Limit: 100

Any line of code must not exceed **100 characters**. Exceptions:

- Lines where obeying the limit would break a package or import statement (see above)
- Command-line strings in comments
- Very long URLs in comments
- Long JSNI method references

### 3.5 Line-Wrapping

When a line exceeds the column limit, it must be broken ("wrapped"). Rules:

#### Break at Higher Syntactic Level

Break after a comma, before an operator, or at a point that minimizes the visual disruption.

```java
// Correct — breaks after comma, continuation +4
someMethod(parameterOne, parameterTwo,
    parameterThree);

// Correct — break before operator
longName1 = longName2 * (longName3 + longName4)
    + longName5 * longName6;
```

#### Continuation Indent: +4

Continuation lines are indented **+4 spaces** from the original line.

```java
return someMethod(longExpression1, longExpression2,
    longExpression3);

String result = someLongMethodName(firstArgument,
    secondArgument, thirdArgument);
```

For method definitions where parameters don't fit:

```java
public SomeLongReturnType someMethodWithLongName(
    String firstParameter, String secondParameter) {
  // body indented +2
}
```

### 3.6 Vertical Whitespace

A single blank line appears:

- Between consecutive members of a class (fields, constructors, methods, inner classes)
- Between logical sections inside a method
- As needed to improve readability

*Never* use multiple blank lines in a row.

```java
public class Example {
  private int count;

  public Example() {
    this.count = 0;
  }

  public int getCount() {
    return count;
  }

  public void setCount(int count) {
    this.count = count;
  }
}
```

### 3.7 Horizontal Whitespace

#### Separating Keywords and Parentheses/Braces

Put a space after `if`, `for`, `while`, `catch`, `synchronized`, `try`, but **not** after the method name.

```java
// Space after keyword
if (condition) { }
for (int i = 0; i < 10; i++) { }
while (condition) { }

// No space after method name
doSomething(argument);
```

#### Binary/Ternary Operators

Put spaces on both sides of binary and ternary operators (except `++`, `--`, `.`, `::`, `->`).

```java
int result = a + b * c;
boolean flag = (x > 0) ? x : -x;
String joined = str1 + str2;
```

#### Type Casts

Put a space after the closing parenthesis.

```java
String value = (String) object;
```

#### Other

- No space before a comma, semicolon, or closing paren/bracket
- One space after a comma, semicolon, colon in `for`-each, or opening brace
- One space before and after `//` in `/* */` and `//` comments

### 3.8 Horizontal Alignment: Not Required

Horizontal alignment (adding spaces to align variable names across lines) is **permitted but not required** and generally discouraged.

```java
// Acceptable but not required
private int    count;
private String name;
```

### 3.9 Grouping Parentheses

Use parentheses to make operator precedence clear, even when not strictly necessary.

```java
// Recommended
return (a + b) * c;

// Also acceptable
if ((a == b) && (c == d)) { }
```

### 3.10 Specific Constructs

#### Enum

Enum constants may be on one line or each on its own line.

```java
// One line
public enum Suit { CLUBS, DIAMONDS, HEARTS, SPADES }

// Multi-line (each constant indented +4, comma-separated)
public enum Suit {
    CLUBS,
    DIAMONDS,
    HEARTS,
    SPADES;
}
```

#### Variable Declarations

One variable per declaration.

```java
// Correct
int x;
int y;

// Forbidden
int x, y;
```

#### Arrays

C-style array declarations are forbidden.

```java
// Correct
String[] args;

// Forbidden
String args[];
```

#### Switch

Braces are always used. Each case block includes a `break` or comment explaining fall-through.

```java
switch (value) {
  case 1:
    handleOne();
    break;
  case 2:
    handleTwo();
    break;
  default:
    handleDefault();
    break;
}
```

Fall-through must be explicitly noted with a comment.

```java
switch (value) {
  case 0:
  case 1:
    // falls through
  case 2:
    handleOneOrTwo();
    break;
}
```

#### Annotations

Annotations that apply to a class, method, or constructor appear on their own lines.

```java
@Override
@Nullable
public String getName() {
  return name;
}
```

Annotations on a single member or type use:

```java
@SuppressWarnings("unchecked")
public <T> T cast(Object obj) {
  return (T) obj;
}
```

Annotations on parameters or local variables go on the same line.

```java
public void process(@Nullable String input) {
  @SuppressWarnings("unused")
  Object temp = input;
}
```

#### Comments

Block comments (`/* ... */`) are indented at the same level as the surrounding code. Line comments (`// ...`) are preferred. Javadoc uses `/** ... */`.

```java
/*
 * This is a block comment (license-like).
 * Wrapped to column limit.
 */
public void example() {
  // Prefer line comments for implementation notes
  doSomething();
}
```

#### Modifiers

Modifiers appear in the order prescribed by the Java Language Specification (recommended order):

```java
public protected private abstract default static final transient volatile synchronized native strictfp
```

#### Numbers

`long` literals use uppercase `L` suffix.

```java
long value = 100L;   // not 100l
```

Hexadecimal, octal, and binary literals may use uppercase or lowercase, but be consistent.

#### Lambda Expressions

Lambdas follow these rules:

```java
// One line, no braces needed
list.forEach(item -> System.out.println(item));

// Multi-line, braces required, explicit return
list.forEach(item -> {
  String upper = item.toUpperCase();
  System.out.println(upper);
});
```

Parentheses around the parameter are required for multiple parameters or when the parameter has a type annotation, optional otherwise.

```java
// Optional parens for single inferred param
list.forEach(item -> process(item));

// Required parens for multiple params
list.forEach((key, value) -> process(key, value));

// Required parens with type annotation
list.forEach((@NotNull String item) -> process(item));
```

#### Text Blocks

Text blocks (Java 13+) are used for multi-line strings.

```java
String json = """
    {
      "name": "John",
      "age": 30
    }
    """;
```

Closing `"""` controls trailing whitespace. Indentation is determined by common leading whitespace.

---

## 4. Naming Deep Dive

### 4.1 Rules Common to All Identifiers

All identifiers must use **only ASCII letters and digits** and, where specified, underscores. Names must match the regex pattern `[a-zA-Z0-9_]+`.

No special prefix or suffix conventions (like `mVar` for member variables, `sVar` for static, or `_var`). Use Hungarian notation **only** where the type system does not provide enough information (rarely needed in modern Java).

### 4.2 Package Names

All lowercase, digits allowed, consecutive underscores are prohibited.

```java
package com.example.deeply.nested;
package com.example.utilities;
```

Underscores should be avoided. If a reserved word is needed, use an underscore:

```java
package com.example.optional_;  // avoids conflict with java.util.Optional
```

### 4.3 Module Names

Module names follow the same rules as package names: all lowercase, no underscores.

```java
module com.example.myapp {
  // ...
}
```

### 4.4 Class Names

UpperCamelCase. Typically nouns or noun phrases.

```java
public class CustomerRepository { }
public class XmlParser { }
public class AbstractFactory { }
public class StringUtils { }
```

Test classes end with `Test`:

```java
public class CustomerRepositoryTest { }
public class XmlParserTest { }
```

### 4.5 Method Names

lowerCamelCase. Typically verbs or verb phrases.

```java
public void execute() { }
public String getUserName() { }
public boolean isEmpty() { }
```

Underscores may appear in JUnit test method names, separating logical components:

```java
@Test
void calculates_total_when_items_added() { }  // allowed in tests only
```

### 4.6 Constants

Constants are `UPPER_SNAKE_CASE`. A constant is a **static final field** whose content is **deeply immutable** and whose methods have no detectable side effects.

```java
static final int MAX_RETRY_COUNT = 3;
static final ImmutableList<String> NAMES = ImmutableList.of("a", "b");
static final ImmutableMap<String, Config> CONFIGS = ImmutableMap.of();
```

Note: `static final` alone does not make a field a constant. If the referenced object can mutate, it is not a constant.

```java
// NOT a constant — object is mutable
static final String[] NAMES = {"a", "b"};

// NOT a constant — list is mutable
static final List<String> NAMES = new ArrayList<>();
```

### 4.7 Non-Constant Fields

lowerCamelCase.

```java
public class Customer {
  private String firstName;
  private String lastName;
  private int age;
}
```

### 4.8 Parameter Names

lowerCamelCase. Public methods should avoid single-character parameter names.

```java
public void sendResponse(HttpResponse response, String message) {
  // ...
}
```

### 4.9 Local Variable Names

lowerCamelCase.

```java
public void process() {
  Customer customer = findCustomer();
  String formattedName = formatName(customer);
  // ...
}
```

### 4.10 Type Variable Names

Each type variable is named as either:

- A single capital letter, optionally followed by a single digit (e.g., `E`, `K`, `V`, `T`, `T1`)
- A full class name-style name ending with `T` (e.g., `RequestT`, `ResponseT`)

Most commonly used:

| Type | Name |
|---|---|
| Element | `E` |
| Key | `K` |
| Value | `V` |
| Map | `K`, `V` |
| General | `T` |
| Exception | `X` or `E` |

### 4.11 Unnamed Variables (Java 21+)

Unused pattern variables or exception parameters use `_`.

```java
try {
  riskyOperation();
} catch (Exception _) {
  System.err.println("Operation failed");
}

switch (obj) {
  case String _ -> handleString();
  case Integer _ -> handleInteger();
  default -> handleDefault();
}
```

### 4.12 Camel Case Defined

The Google Java Style Guide contains a precise definition of how to convert names to camel case. The algorithm:

1. Start with the prose form (the name as a phrase).
2. Convert the phrase to plain ASCII and remove any apostrophes.
3. Split the result into words, splitting on spaces and any remaining punctuation (typically hyphens).
4. Now "lowercase" everything, and then uppercase only the first character of each word to produce UpperCamelCase.
5. For lowerCamelCase, lowercase the first character of the first word.

**Acronyms are treated as whole words** — not as strings of individual capitals.

| Prose form | UpperCamelCase | lowerCamelCase |
|---|---|---|
| XML HTTP request | `XmlHttpRequest` | `xmlHttpRequest` |
| new customer ID | `NewCustomerId` | `newCustomerId` |
| supports SSL | `SupportsSsl` | `supportsSsl` |
| old HTML document | `OldHtmlDocument` | `oldHtmlDocument` |
| parse DNS record | `ParseDnsRecord` | `parseDnsRecord` |
| SQL database | `SqlDatabase` | `sqlDatabase` |
| HTTP connection | `HttpConnection` | `httpConnection` |
| URL helper | `UrlHelper` | `urlHelper` |
| IO stream | `IoStream` | `ioStream` |

Note the pattern: `XML` becomes `Xml`, `HTTP` becomes `Http`, `URL` becomes `Url`, `IO` becomes `Io` — in camel case, only the first letter is capitalized.

---

## 5. Javadoc Deep Dive

### 5.1 General Form

Javadoc block comments begin with `/**` and end with `*/`. Every Javadoc block starts with a **summary fragment** (a noun phrase or verb phrase, not a full sentence), followed by block tags.

```java
/**
 * A customer record with name and contact information.
 */
public class Customer {
  /**
   * Returns the display name for this customer.
   *
   * @return the display name, never null
   */
  public String getDisplayName() {
    return displayName;
  }
}
```

### 5.2 Block Tags

Block tags appear after the description, in the order: `@param`, `@return`, `@throws`, `@see`, `@since`, `@deprecated`. No blank line between a tag and its description.

```java
/**
 * Processes the given input file.
 *
 * @param inputFile the file to process; must exist and be readable
 * @param encoding  the character encoding to use, e.g. "UTF-8"
 * @return the processing result
 * @throws IOException if reading or writing fails
 * @since 2.0
 */
public Result process(File inputFile, String encoding) throws IOException {
  // ...
}
```

### 5.3 Standalone Paragraph Tags

Use `<p>` tags to separate paragraphs within the description. Do NOT use `</p>`; `<p>` is a standalone tag.

```java
/**
 * A builder for constructing {@link Foo} instances.
 *
 * <p>This builder is not thread-safe. Each thread should create
 * its own instance.
 *
 * <p>Example usage:
 * <pre>{@code
 *   Foo foo = Foo.builder()
 *       .setName("example")
 *       .build();
 * }</pre>
 */
```

### 5.4 Summary Fragment

The first block of Javadoc text (up to the first period followed by whitespace or block tag) forms the summary fragment. It must be a concise noun or verb phrase, not a full sentence.

```java
// Good (verb phrase)
/** Returns the customer name. */

// Good (noun phrase)
/** A customer record. */

// Not ideal (full sentence)
/** This method returns the customer name. */
```

### 5.5 Where Javadoc Is Required

#### Classes

Every **public or protected** class (including interfaces, enums, annotations) must have Javadoc.

```java
/**
 * Provides database access operations for the customer domain.
 */
public class CustomerDao {
  // ...
}
```

Exception: A class whose purpose is obvious from its name, such as utility classes with only `main()`.

#### Methods

Every **public or protected** method must have Javadoc.

```java
/**
 * Calculates the total price including tax.
 *
 * @param basePrice the base price before tax
 * @param taxRate   the tax rate as a decimal (e.g. 0.08 for 8%)
 * @return the total price including tax
 */
public BigDecimal calculateTotal(BigDecimal basePrice, BigDecimal taxRate) {
  // ...
}
```

Exceptions:
- Simple getters/setters (`getX()`, `setX(val)`)
- Overrides of abstract methods (Javadoc is inherited)
- Trivial one-liners

#### Fields

Constants (`static final` deeply immutable fields) require Javadoc.

```java
/** The default page size for paginated results. */
static final int DEFAULT_PAGE_SIZE = 20;
```

Non-constant fields generally do not require Javadoc unless they have complex semantics.

### 5.6 Javadoc Paragraphs

Each paragraph after the first begins with `<p>` on its own line.

```java
/**
 * A thread-safe mutable counter.
 *
 * <p>This class provides atomic increment and decrement operations.
 *
 * <p>Instances are not serializable.
 */
```

HTML tags like `<pre>`, `<code>`, `<ul>`, `<li>` may be used inside Javadoc.

### 5.7 @param

Every parameter of a method must have a `@param` tag.

```java
/**
 * @param query    the search query string; must not be empty
 * @param maxResults the maximum number of results to return (1-100)
 */
```

### 5.8 @return

If the method returns something (not `void`), it must have `@return`.

```java
/** @return the computed result, never null */
```

### 5.9 @throws

Every declared checked exception and every documented unchecked exception has a `@throws` tag.

```java
/**
 * @throws IOException if the underlying stream cannot be read
 * @throws IllegalArgumentException if the input is malformed
 */
```

### 5.10 @since

Documents when the API element was introduced.

```java
/** @since 2.0 */
```

### 5.11 @deprecated

Documents that the API element should no longer be used, with a reason and replacement.

```java
/**
 * Returns the old-style identifier.
 *
 * @deprecated Use {@link #getNewIdentifier()} instead. This method
 *     will be removed in version 3.0.
 */
@Deprecated
public String getOldId() {
  return oldId;
}
```

---

## 6. Review Checklist

Use the following checklist when writing or reviewing Java code for Google style compliance:

### Naming
- [ ] Package names are all lowercase, no underscores
- [ ] Class names are UpperCamelCase (acronyms as whole words)
- [ ] Method names are lowerCamelCase
- [ ] Constants are UPPER_SNAKE_CASE and are deeply immutable
- [ ] Non-constant fields are lowerCamelCase
- [ ] Parameters are lowerCamelCase
- [ ] Type variables are single capital letter or NameT

### Formatting
- [ ] Column limit is 100 characters
- [ ] Indentation is 2 spaces (no tabs)
- [ ] K&R brace style throughout
- [ ] Braces used even for single-line blocks
- [ ] Continuation indent is +4 spaces
- [ ] Line wrapping at higher syntactic level
- [ ] One statement per line

### Source Structure
- [ ] File name matches top-level class name
- [ ] UTF-8 encoding
- [ ] No wildcard imports
- [ ] Imports in ASCII order, no line-wrapping
- [ ] Exactly one top-level class per file
- [ ] Overloaded methods grouped together

### Annotations
- [ ] `@Override` always used when overriding
- [ ] `@Nullable` / `@NonNull` used for nullable/non-null parameters and returns

### Language
- [ ] No C-style array declarations (`String args[]` is forbidden)
- [ ] No wildcard imports
- [ ] Prefer immutable collections (`ImmutableList`, `ImmutableMap`)
- [ ] Acronyms treated as whole words in camel case (XmlHttpRequest, not XMLHTTPRequest)
- [ ] Long literals use uppercase `L`

### Javadoc
- [ ] All public/protected classes have Javadoc
- [ ] All public/protected methods have Javadoc (except simple getters/setters)
- [ ] All parameters have `@param`
- [ ] Non-void methods have `@return`
- [ ] All thrown exceptions have `@throws`
- [ ] Summary fragment is a phrase, not a full sentence
- [ ] `<p>` used for paragraph breaks (standalone, no `</p>`)
- [ ] Deprecated items have `@deprecated` with reason and replacement

---

Full source: `G:\styleguide\javaguide.html`
