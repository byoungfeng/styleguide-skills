---
name: google-style-ts-deep
description: Comprehensive deep reference for the Google TypeScript Style Guide covering source file basics, structure, formatting, naming, type system, and language features.
---

# Google TypeScript Style Guide — Deep Reference

## 1. Source File Basics

### 1.1 File Encoding

All source files must be encoded in **UTF-8**. The UTF-8 byte order mark (BOM) is prohibited.

### 1.2 Whitespace Characters

- The horizontal whitespace character is the **space** (0x20)
- The **tab character** (0x09) is prohibited for indentation — use 2 spaces
- The end-of-line sequence is **LF** (0x0A) — even on Windows
- No trailing whitespace at the end of lines

### 1.3 Special Escape Sequences

For characters with special escape sequences (`\'`, `\"`, `\\`, `\n`, `\r`, `\t`, `\b`, `\f`, `\v`, `\0`), use the escape sequence rather than the literal character:

```typescript
// Good
const msg = 'He said \"hello\"';
const nl = '\n';

// Bad
const msg = 'He said "hello"';
const nl = '\u000a';
```

### 1.4 Non-ASCII Characters

Non-ASCII characters are allowed in string literals, comments, and documentation. Use Unicode escapes (`\uXXXX` or `\u{XXXXX}`) only when required by tooling limitations. Prefer the actual Unicode character when possible:

```typescript
// Good
const units = 'μs';
const copyright = '©';

// Acceptable
const units = '\u03bcs';
```

## 2. Source File Structure

### 2.1 Overall Order

Files consist of, in order:

1. **Copyright and license** (if any) — as a banner comment
2. **`@fileoverview` JSDoc** — optional, only when the file requires explanation beyond its name
3. **Imports** — all `import` statements
4. **Implementation** — the actual code

```typescript
/**
 * @fileoverview Description of file contents and usage.
 */

import { Foo } from './foo';
import type { Bar } from './bar';

class MyClass { /* ... */ }
export default MyClass;
```

### 2.2 Copyright / License

If a copyright header is required by the project, it goes as the very first lines in the file. Use a `//`-style comment banner or block comment:

```typescript
// Copyright 2024 The Example Company
//
// Licensed under the Apache License, Version 2.0 (the "License");
// ...
```

### 2.3 `@fileoverview` JSDoc

Use a `@fileoverview` block only when the file's purpose is not obvious from its name. Place it immediately after any copyright header:

```typescript
/**
 * @fileoverview Utilities for date arithmetic and formatting.
 */
```

### 2.4 Imports

#### 2.4.1 Import Types

| Import Type | Syntax | When to Use |
|---|---|---|
| Module alias | `import * as lib from './lib'` | Importing a library module |
| Named | `import { Foo } from './foo'` | Importing specific exports |
| Default | `import Foo from './foo'` | **Avoid** — only for legacy interop |
| Side-effect | `import './polyfills'` | For polyfills / setup code |

#### 2.4.2 Import Paths

- **Project code**: relative paths (`./foo`, `../bar/baz`)
- **External packages**: bare specifiers (`@angular/core`, `lodash`)
- No file extensions in import paths (`./foo` not `./foo.ts`)

#### 2.4.3 Namespace vs Named Imports

Use named imports (`import { Foo } from './foo'`) over namespace imports (`import * as foo from './foo'`) when you only need a subset of exports. Use namespace imports when importing an entire module that acts as a namespace (e.g., utility libraries with many functions):

```typescript
// Good — small selection
import { parseDate, formatDate } from './date-utils';

// Good — entire module used as namespace
import * as dateUtils from './date-utils';

// Bad — namespace import just to avoid named import
import * as foo from './foo';
const bar = foo.bar;  // prefer: import { bar } from './foo'
```

#### 2.4.4 Renaming Imports

Rename imports to avoid naming conflicts. Use the `as` keyword:

```typescript
import { Button as MaterialButton } from '@material/button';
import { Button as CustomButton } from './button';
```

#### 2.4.5 Type-Only Imports

Use `import type` for imports that are used **only** as types (interfaces, type aliases, etc.). This allows the transpiler to elide the import entirely:

```typescript
import type { User, UserConfig } from './user';
import { createUser } from './user';
```

#### 2.4.6 Prohibited Import Constructs

No `require()`:
```typescript
// Bad
const fs = require('fs');
// Good
import * as fs from 'fs';
```

No `namespace`:
```typescript
// Bad
namespace MyNamespace { /* ... */ }
```

No `/// <reference>` directives:
```typescript
// Bad
/// <reference path="foo.d.ts" />
// Good
import type { Foo } from './foo';
```

### 2.5 Exports

#### 2.5.1 Named Exports Only

Prefer named exports over default exports:

```typescript
// Good
export class UserService { /* ... */ }
export function formatName() { /* ... */ }

// Bad
export default class UserService { /* ... */ }
```

Named exports are more discoverable, support renaming at import time, and enable better tooling.

#### 2.5.2 Export Visibility

Export only what consumers need. Keep internal helpers unexported:

```typescript
// Internal — not exported
function formatInternalDate(date: Date): string { /* ... */ }

// Public API — exported
export function formatDisplayDate(date: Date): string {
  return formatInternalDate(date);
}
```

#### 2.5.3 Mutable Exports

Do not export mutable bindings. Exported values should be treated as constants:

```typescript
// Bad
export let currentUser: User | null = null;

// Good
let currentUser: User | null = null;
export function getCurrentUser(): User | null {
  return currentUser;
}
```

#### 2.5.4 Container Classes

Do not create container classes whose sole purpose is to group static methods:

```typescript
// Bad
export class StringUtils {
  static capitalize(s: string): string { /* ... */ }
  static trim(s: string): string { /* ... */ }
}

// Good
export function capitalize(s: string): string { /* ... */ }
export function trim(s: string): string { /* ... */ }
```

## 3. Formatting Deep Dive

### 3.1 Braces (K&R Style)

Opening brace on the same line as the statement, closing brace on its own line:

```typescript
// Good
class MyClass {
  method() {
    if (condition) {
      // ...
    } else {
      // ...
    }
  }
}

// Bad
class MyClass
{
  method()
  {
    if (condition)
    {
      // ...
    }
  }
}
```

Braces are **required** for all control structures (`if`, `else`, `for`, `do`, `while`), even when the body is a single statement:

```typescript
// Good
if (condition) {
  return;
}

// Bad
if (condition) return;
```

### 3.2 Block Indentation (+2)

Each new block or brace-delimited construct adds 2 spaces of indentation:

```typescript
class Foo {
  constructor(public name: string) {}

  greet(): string {
    if (this.name) {
      return `Hello, ${this.name}`;
    }
    return 'Hello, world';
  }
}
```

### 3.3 Column Limit: 80

Each line of code must be at most 80 characters. Strings that exceed 80 chars should use concatenation or template literals:

```typescript
// Good
const msg = 'This is a very long string that exceeds ' +
    'the 80-character column limit so it is broken up.';

// Also good (template literal)
const msg = `This is a very long string that exceeds
    the 80-character column limit so it is broken up.`;
```

### 3.4 Line-Wrapping

When a line must be broken at the 80-char limit, break at a **higher syntactic level**:

- Break after an operator (not before)
- Break after a comma (not before)
- Align the continuation line with +4 spaces from the start of the expression:

```typescript
// Good
const result = someFunctionWithALongName(argumentOne, argumentTwo,
    argumentThree, argumentFour);

// Bad — break at lower level
const result = someFunctionWithALongName(argumentOne, argumentTwo, argumentThree,
  argumentFour);
```

### 3.5 Continuation Indent (+4)

When line-wrapping, continuation lines use **+4 spaces** (not +2):

```typescript
// Good
return {
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
  roles: ['admin',
      'editor',
      'viewer'],
};

// Bad — continuation indent same as block indent
return {
  roles: ['admin',
    'editor',
    'viewer'],
};
```

### 3.6 Blank Lines

- One blank line between top-level constructs (classes, functions, constants)
- One blank line between import blocks and implementation
- No blank lines after `{` or before `}`
- One blank line between methods in a class (optional for one-liners)

```typescript
import { Foo } from './foo';
import { Bar } from './bar';

const MAX_COUNT = 100;

export class MyClass {
  private foo: Foo;

  constructor(foo: Foo) {
    this.foo = foo;
  }

  doSomething(): void {
    // ...
  }
}

function helper(): void {
  // ...
}
```

### 3.7 Horizontal Whitespace

Place a space on both sides of binary and ternary operators:

```typescript
// Good
const sum = a + b;
const result = condition ? x : y;
const index = i + j * 2;

// Bad
const sum = a+b;
const result = condition?x:y;
const index = i+j*2;
```

No space after unary operators:

```typescript
// Good
const neg = -value;
const inc = ++count;

// Bad
const neg = - value;
const inc = ++ count;
```

Space after `//` and before `//` comments:

```typescript
// Good
const x = 5;  // this is a comment

// Bad
const x = 5; //this is a comment
```

Space between control keyword and opening parenthesis:

```typescript
// Good
if (condition) { ... }
while (condition) { ... }

// Bad
if(condition) { ... }
while(condition) { ... }
```

No space between function name and opening parenthesis:

```typescript
// Good
function foo() { ... }
const result = foo();

// Bad
function foo () { ... }
const result = foo ();
```

Space after comma in argument lists, array literals, and object literals:

```typescript
// Good
const arr = [1, 2, 3];
const obj = { a: 1, b: 2 };
foo(a, b, c);

// Bad
const arr = [1,2,3];
const obj = { a:1, b:2 };
foo(a,b,c);
```

### 3.8 Horizontal Alignment (Not Required)

Do not attempt to align variable declarations, assignments, or type annotations vertically:

```typescript
// Good (not required)
const name        = 'Alice';
const age         = 30;
const occupation  = 'Engineer';

// Preferred
const name = 'Alice';
const age = 30;
const occupation = 'Engineer';
```

### 3.9 Specific Constructs

#### 3.9.1 Arrays

Use array literals. No `Array()` constructor:

```typescript
// Good
const items: string[] = [];
const numbers = [1, 2, 3];

// Bad
const items = new Array<string>();
const numbers = new Array(1, 2, 3);
```

Array destructuring for extracting values:

```typescript
const [first, second, ...rest] = [1, 2, 3, 4, 5];
```

#### 3.9.2 Objects

Use object literals. Quote property names only when necessary:

```typescript
// Good
const obj = { name: 'Alice', age: 30 };

// Good — quotes required due to special chars
const obj = { 'first-name': 'Alice', 'class': 'admin' };

// Bad — unnecessary quotes
const obj = { 'name': 'Alice', 'age': 30 };
```

Shorthand property and method names:

```typescript
// Good
const name = 'Alice';
const obj = { name };

const obj2 = {
  greet() {
    return 'Hello';
  },
};
```

Object destructuring (use only the properties you need):

```typescript
const { name, age } = getUser();
const { length: len } = 'hello';
```

#### 3.9.3 Classes

```typescript
export class User {
  private readonly id: string;
  private name: string;

  constructor(id: string, name: string) {
    this.id = id;
    this.name = name;
  }

  getName(): string {
    return this.name;
  }

  setName(name: string): void {
    this.name = name;
  }
}
```

Parameter properties allowed (combine declaration and assignment):

```typescript
export class User {
  constructor(
    private readonly id: string,
    public name: string,
  ) {}
}
```

#### 3.9.4 Functions

```typescript
// Named function
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function assigned to const
const multiply = (a: number, b: number): number => a * b;

// Function expression
const divide = function (a: number, b: number): number {
  return a / b;
};
```

Functions with many parameters — one per line:

```typescript
function createUser(
    id: string,
    name: string,
    email: string,
    roles: string[],
): User {
  // ...
}
```

#### 3.9.5 String Literals

Use single quotes by default. Use double quotes for JSON or strings containing single quotes:

```typescript
// Good
const name = 'Alice';
const msg = "It's done";

// Template literals for interpolation
const greeting = `Hello, ${name}!`;
```

Template literals are preferred over concatenation:

```typescript
// Good
const url = `${protocol}://${host}/${path}`;

// Bad
const url = protocol + '://' + host + '/' + path;
```

#### 3.9.6 Boolean and Number Literals

Use lowercase `true`, `false`, `null`, `undefined`:

```typescript
// Good
const isActive = true;
const value = null;
const notDefined = undefined;

// Bad
const isActive = True;
const value = NULL;
```

Use numeric separators for readability with large numbers:

```typescript
const billion = 1_000_000_000;
const hexMask = 0xFF_FF_FF_FF;
```

#### 3.9.7 Control Structures

**if/else:**

```typescript
if (condition) {
  // ...
} else if (otherCondition) {
  // ...
} else {
  // ...
}
```

**for/of (preferred over for/in):**

```typescript
// Good — iterating values
for (const item of items) {
  // ...
}

// Good — with index
for (const [i, item] of items.entries()) {
  // ...
}

// Bad — for...in (iterates keys, includes prototype)
for (const key in obj) {
  if (Object.prototype.hasOwnProperty.call(obj, key)) {
    // ...
  }
}
```

**while/do:**

```typescript
while (condition) {
  // ...
}

do {
  // ...
} while (condition);
```

**switch:**

```typescript
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

**try/catch:**

```typescript
try {
  doSomethingRisky();
} catch (err) {
  handleError(err);
} finally {
  cleanup();
}
```

## 4. Naming Deep Dive

### 4.1 Identifiers

All identifiers must be ASCII only. Use only the characters `A-Z`, `a-z`, `0-9`, and `_` (underscore, limited use). Unicode characters in identifiers are prohibited:

```typescript
// Bad
const π = 3.14159;
const tamaño = 'large';

// Good
const pi = 3.14159;
const size = 'large';
```

### 4.2 Naming Style — Prohibited Patterns

**No Hungarian notation:** Do not encode type information in variable names:

```typescript
// Bad
const strName = 'Alice';
const iCount = 5;
const arrItems: string[] = [];

// Good
const name = 'Alice';
const count = 5;
const items: string[] = [];
```

**No `I` prefix for interfaces:**

```typescript
// Bad
interface IUser { /* ... */ }
interface IShape { /* ... */ }

// Good
interface User { /* ... */ }
interface Shape { /* ... */ }
```

**No `_` prefix or suffix** (even for private members):

```typescript
// Bad
class Foo {
  private _internal: string;
  private internal_: string;
}

// Good
class Foo {
  private internal: string;
}
```

**No `_` for unused parameters** — use `_` as an entire parameter name:

```typescript
// Bad
function process(unusedParam: string, usedParam: number) {}

// Good
function process(_: string, usedParam: number) {}
```

### 4.3 Descriptive Names

Be descriptive. Avoid abbreviations that are not widely understood:

```typescript
// Good
const errorMessage: string;
const isLoading: boolean;
const userCount: number;

// Bad
const errMsg: string;    // ambiguous abbreviation
const loadFlg: boolean;  // cryptic abbreviation
const usrCnt: number;    // unnecessary shortening
```

Acceptable abbreviations include industry-standard ones (`i`, `j`, `n`, `x`, `y`, `z` for loop indices; `e` for event; `evt` for event; `fn` for function; `str` for string; `num` for number):

```typescript
for (let i = 0; i < n; i++) { /* ... */ }
element.addEventListener('click', (e: MouseEvent) => { /* ... */ });
```

### 4.4 Camel Case with Acronyms

Treat acronyms as words for camel-casing purposes:

```typescript
// Good
const loadHttpUrl: string;
const parseXmlString: string;
const htmlParser: HtmlParser;
const dbConnection: DatabaseConnection;

// Bad
const loadHTTPURL: string;
const parseXMLString: string;
const HTMLParser: HtmlParser;
const DBConnection: DatabaseConnection;
```

### 4.5 Rules by Identifier Type

| Identifier Type | Style | Examples |
|---|---|---|
| Class | UpperCamelCase | `class UserService` |
| Interface | UpperCamelCase | `interface UserConfig` |
| Type alias | UpperCamelCase | `type JsonValue` |
| Enum | UpperCamelCase | `enum Color` |
| Enum value | CONSTANT_CASE | `RED`, `GREEN`, `BLUE` |
| Decorator | UpperCamelCase | `@Component`, `@Injectable` |
| Type parameter | UpperCamelCase | `<T>`, `<K, V>` |
| Variable | lowerCamelCase | `const userCount = 5` |
| Parameter | lowerCamelCase | `(name: string, age: number)` |
| Function | lowerCamelCase | `function getUser()` |
| Method | lowerCamelCase | `getName()`, `setName()` |
| Property | lowerCamelCase | `private name: string` |
| Module alias | lowerCamelCase | `import * as express from 'express'` |
| Global constant | CONSTANT_CASE | `const MAX_RETRIES = 3` |
| Enum value | CONSTANT_CASE | `Color.RED` |
| Test name | backticked sentences | `` it(`should return true when condition met`) `` |
| Local constant | lowerCamelCase | `const maxItems = 10` |

### 4.6 Type Parameters

Single uppercase letter by convention, or a descriptive UpperCamelCase name:

```typescript
// Simple
function identity<T>(arg: T): T {
  return arg;
}

// Descriptive
class KeyValueMap<K, V> {
  private map = new Map<K, V>();
}

// Common conventions
<T>     // generic type
<K, V>  // key, value
<E>     // element type
<R>     // return type
<P>     // props / parameters
```

### 4.7 Test Names

Test names should be descriptive sentences in backticks (when using the test framework that supports it) or use descriptive lowerCamelCase:

```typescript
describe('UserService', () => {
  it(`should return user when valid id is provided`, () => {
    // ...
  });

  it(`should throw when user is not found`, () => {
    // ...
  });
});
```

### 4.8 Constants: Global vs Local

**Global constants** (module-level, exported or used broadly) use `CONSTANT_CASE`:

```typescript
export const MAX_FILE_SIZE = 10_485_760;  // 10 MB
const DEFAULT_TIMEOUT = 5000;              // 5 seconds
```

**Local constants** (function-level or block-scoped) use `lowerCamelCase`:

```typescript
function processFile(path: string): void {
  const maxAttempts = 3;
  const retryDelay = 1000;
  // ...
}
```

### 4.9 Aliases

When creating a local alias for an imported or existing name, use the same casing convention as the original:

```typescript
import { UserService as UserServiceAlias } from './user-service';

const myHandler = someLongNamedFunction;
```

## 5. Type System Deep Dive

### 5.1 Type Inference

Prefer letting TypeScript infer types. Only add explicit annotations when inference is insufficient:

```typescript
// Good — inferred
const name = 'Alice';
const count = 42;
const items = ['a', 'b', 'c'];

// When needed — explicit
const parsed: Record<string, unknown> = JSON.parse(jsonString);
function fetchUser(id: string): Promise<User> { /* ... */ }
```

### 5.2 Return Types

Function return types are optional but recommended for exported functions and complex logic:

```typescript
// Good practice for exported functions
export function add(x: number, y: number): number {
  return x + y;
}

// Acceptable for simple internal functions
function add(x: number, y: number) {
  return x + y;
}
```

### 5.3 `undefined` and `null`

- Use `undefined` (not `null`) for absent or uninitialized values
- Avoid `null` unless interfacing with an API that requires it
- Use optional `?` syntax rather than `| undefined`

```typescript
// Good
function getUser(id: string): User | undefined {
  // ...
}

// Good — optional parameter
function createUser(name: string, email?: string): User {
  // ...
}

// Bad — using null
function getUser(id: string): User | null {
  // ...
}
```

### 5.4 Prefer Optional `?` Over `| undefined`

```typescript
// Good
interface Config {
  timeout?: number;
  retries?: number;
}

// Bad
interface Config {
  timeout: number | undefined;
  retries: number | undefined;
}
```

### 5.5 Use Structural Types

TypeScript uses structural typing (duck typing). Prefer it over nominal patterns:

```typescript
interface HasLength {
  length: number;
}

function logLength(obj: HasLength): void {
  console.log(obj.length);
}

// Works with any object that has `.length`
logLength('hello');        // string has length
logLength([1, 2, 3]);     // array has length
logLength({ length: 10 }); // plain object with length
```

### 5.6 Use `unknown` Not `any`

When a value's type is truly unknown at write time, use `unknown` instead of `any`. This forces type-checking before use:

```typescript
// Good
function parseData(json: string): unknown {
  return JSON.parse(json);
}

const data = parseData('{"name":"Alice"}');
// Must narrow before use
if (typeof data === 'object' && data !== null && 'name' in data) {
  console.log((data as Record<string, unknown>).name);
}

// Bad — any loses all type safety
function parseData(json: string): any {
  return JSON.parse(json);
}
const data = parseData('{"name":"Alice"}');
console.log(data.name);  // no type checking
```

### 5.7 Prefer `readonly`

Mark properties, parameters, and array types as `readonly` to enforce immutability:

```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

function processItems(items: readonly string[]): void {
  // Cannot modify items
  // items.push('new');  // Error
  // items[0] = 'a';     // Error
}
```

Use `ReadonlyArray<T>` or `readonly T[]` for read-only arrays. Use `Readonly<T>` for object types:

```typescript
function process(config: Readonly<AppConfig>): void {
  // config is immutable
}
```

### 5.8 Type Assertions

Use `as` syntax for type assertions (not angle-bracket syntax, which conflicts with JSX):

```typescript
// Good
const el = document.getElementById('root') as HTMLDivElement;
const value = data as Record<string, unknown>;

// Bad — angle-bracket syntax
const el = <HTMLDivElement>document.getElementById('root');
```

Use the `!` (non-null assertion) operator sparingly — only when you are certain a value is not `null`/`undefined`:

```typescript
const name = user!.name;  // Only if you know user is non-null
```

Prefer conditional checks over assertions:

```typescript
// Better
if (user) {
  const name = user.name;
}
```

### 5.9 Enum Usage

Prefer `const enum` or string literal unions over regular enums when possible:

```typescript
// Good — string literal union (no runtime cost)
type Color = 'red' | 'green' | 'blue';

// Good — const enum
const enum Color {
  RED = 'red',
  GREEN = 'green',
  BLUE = 'blue',
}

// Regular enum (when you need runtime object)
enum HttpStatus {
  OK = 200,
  NOT_FOUND = 404,
  INTERNAL_ERROR = 500,
}
```

### 5.10 `@`-sign for Accessing Fields

When accessing a property with a special name (e.g., a private field or a symbol-keyed property), use the `@` pattern only in JSDoc references:

```typescript
class Foo {
  private bar = 1;
  // In JSDoc: {@link Foo#bar}
}
```

For dynamic property access, use bracket notation:

```typescript
const key = 'name';
const value = obj[key];
```

### 5.11 `const` Assertions

Use `as const` to narrow literal types and make properties `readonly`:

```typescript
// Without const assertion
const config = { host: 'localhost', port: 8080 };
// config.host is type 'string', config.port is type 'number'

// With const assertion
const config = { host: 'localhost', port: 8080 } as const;
// config.host is type 'localhost', config.port is type 8080
// All properties are readonly
```

### 5.12 `keyof` / `typeof`

Use `keyof` to get the union of property keys of a type:

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

type UserKeys = keyof User;  // 'name' | 'age' | 'email'

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

Use `typeof` to capture the type of a value:

```typescript
const user = { name: 'Alice', age: 30 };
type User = typeof user;  // { name: string; age: number; }
```

## 6. Language Features Deep Dive

### 6.1 Local Variable Declarations

Use `const` for all variables that are not reassigned. Use `let` only when reassignment is necessary. Never use `var`:

```typescript
// Good
const maxRetries = 3;
let currentAttempt = 0;
while (currentAttempt < maxRetries) {
  // ...
  currentAttempt++;
}

// Bad
var maxRetries = 3;
```

One declaration per variable:

```typescript
// Good
const a = 1;
const b = 2;

// Bad
const a = 1, b = 2;
```

Declare variables as close to their first use as possible (not at the top of a function):

```typescript
function processItems(items: string[]): string[] {
  const result: string[] = [];
  // ... some code ...
  for (const item of items) {
    const transformed = transform(item);  // declared close to use
    result.push(transformed);
  }
  return result;
}
```

### 6.2 Array and Object Literals

Always use literal syntax:

```typescript
// Arrays
const arr = [1, 2, 3];        // Good
const arr = new Array(1, 2, 3);  // Bad
const arr = new Array(3);        // Bad (creates [empty × 3])

// Objects
const obj = { a: 1, b: 2 };   // Good
const obj = new Object();      // Bad
const obj = {};                // Good (empty object)
```

Use spread operator for copying:

```typescript
const original = [1, 2, 3];
const copy = [...original];

const originalObj = { a: 1, b: 2 };
const merged = { ...originalObj, c: 3 };
```

### 6.3 String, Boolean, and Number Literals

String literals use single quotes. Boolean and number literals use lowercase:

```typescript
const s = 'hello';
const b = true;
const n = 42;
const nl = null;
const undef = undefined;
```

Use `String()` for explicit conversion (not `.toString()` which can throw on `null`/`undefined`):

```typescript
const value: unknown = getUserInput();
const str = String(value);  // Safe
// const str = value.toString();  // Unsafe — throws if value is null/undefined
```

### 6.4 Control Structures

All control structures must use braces:

```typescript
// Good
if (condition) {
  doSomething();
}

// Bad
if (condition) doSomething();
```

No `for...in` — use `for...of`:

```typescript
// Good
for (const item of items) {
  // ...
}

// Bad
for (const key in items) {
  // ...
}
```

Use `for...of` with `.entries()` when you need the index:

```typescript
for (const [index, item] of items.entries()) {
  console.log(index, item);
}
```

Use `switch` sparingly; prefer object maps or discriminated unions:

```typescript
// Preferred over switch
type Action = 'create' | 'update' | 'delete';

const handlers: Record<Action, () => void> = {
  create: handleCreate,
  update: handleUpdate,
  delete: handleDelete,
};

handlers[action]();
```

### 6.5 Arrow Functions

Arrow functions are preferred for callbacks because they capture `this` lexically:

```typescript
// Good
items.forEach((item) => {
  console.log(item);
});

// Good — concise body
const doubled = items.map((x) => x * 2);

// Avoid function expression for callbacks
items.forEach(function (item) {
  console.log(item);
});
```

Parentheses around parameters:

```typescript
// Zero parameters
const fn = () => { /* ... */ };

// One parameter (parentheses optional)
const fn = (x: number) => x * 2;

// Multiple parameters
const fn = (a: number, b: number) => a + b;
```

### 6.6 `this`

Only use `this` in class methods, arrow functions within classes, or when explicitly documented. Avoid `that = this` patterns — use arrow functions instead:

```typescript
// Good
class MyClass {
  private value = 42;

  method(): void {
    setTimeout(() => {
      console.log(this.value);  // lexical this
    }, 1000);
  }
}

// Bad — that = this
class MyClass {
  private value = 42;

  method(): void {
    const that = this;
    setTimeout(function () {
      console.log(that.value);
    }, 1000);
  }
}
```

### 6.7 Equality Checks

Always use `===` and `!==`. Never use `==` or `!=`:

```typescript
// Good
if (value === null) { /* ... */ }
if (value !== undefined) { /* ... */ }
if (count === 0) { /* ... */ }

// Bad — loose equality
if (value == null) { /* ... */ }
if (value != undefined) { /* ... */ }
```

### 6.8 Classes

Classes should follow these conventions:

```typescript
export class UserRepository {
  private readonly users = new Map<string, User>();

  constructor(private readonly db: Database) {}

  async findById(id: string): Promise<User | undefined> {
    return this.users.get(id);
  }

  async save(user: User): Promise<void> {
    this.users.set(user.id, user);
    await this.db.persist(user);
  }
}
```

- Use `private` / `public` / `protected` modifiers explicitly
- Use `readonly` for fields set once in the constructor
- Parameter properties are allowed
- No `_` prefix for private members — trust the `private` keyword
- No container classes (classes with only static members)

### 6.9 Interfaces

Prefer `interface` over `type` for object shapes:

```typescript
// Good
interface User {
  id: string;
  name: string;
  email: string;
}

// Acceptable — type for unions, primitives, tuples
type Status = 'active' | 'inactive';
type Pair<T> = [T, T];
type Callback = (err: Error | null, result?: unknown) => void;
```

Interfaces support declaration merging, which can be useful:

```typescript
interface Window {
  myCustomProperty: string;
}

// Later, this merges with the previous declaration
interface Window {
  anotherProperty: number;
}
```

### 6.10 Generics

Prefer descriptive type parameter names for non-trivial generics:

```typescript
// Simple — single letter ok
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

// Complex — descriptive names
class Cache<Key extends string, Value> {
  private store = new Map<Key, Value>();

  get(key: Key): Value | undefined {
    return this.store.get(key);
  }

  set(key: Key, value: Value): void {
    this.store.set(key, value);
  }
}
```

Use generic constraints with `extends` to restrict type parameters:

```typescript
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find((item) => item.id === id);
}
```

### 6.11 Modules

Use ES6 module syntax exclusively:

```typescript
// Export
export function publicFn(): void { /* ... */ }
export class PublicClass { /* ... */ }
export interface PublicInterface { /* ... */ }

// Import
import { publicFn, PublicClass } from './module';
import type { PublicInterface } from './module';
```

Avoid circular dependencies. If A imports from B and B imports from A, extract shared types into a third module:

```typescript
// shared-types.ts — extracted to break circular dependency
export interface SharedType { /* ... */ }

// a.ts
import type { SharedType } from './shared-types';

// b.ts
import type { SharedType } from './shared-types';
```

## 7. Review Checklist

When reviewing TypeScript code for Google style guide compliance, check:

### Naming
- [ ] All identifiers use correct casing per identifier type
- [ ] No `I` prefix on interfaces
- [ ] No `_` prefix/suffix on identifiers (including private members)
- [ ] No Hungarian notation
- [ ] Abbreviations treated as words in camelCase
- [ ] Global constants use `CONSTANT_CASE`

### Formatting
- [ ] No line exceeds 80 characters
- [ ] 2-space indentation, no tabs
- [ ] K&R brace style on all control structures and classes
- [ ] Continuation indent is +4 spaces
- [ ] Semicolons present
- [ ] No trailing whitespace
- [ ] One blank line between top-level constructs

### Imports
- [ ] ES6 `import`/`export` syntax — no `require()`, no `namespace`, no `<reference>`
- [ ] `import type` for type-only imports
- [ ] No default exports (prefer named exports)
- [ ] No circular dependencies
- [ ] Relative paths for project code

### Type System
- [ ] Type inference preferred over explicit annotations
- [ ] `unknown` used instead of `any`
- [ ] Optional `?` preferred over `| undefined`
- [ ] `undefined` preferred over `null`
- [ ] `readonly` used where appropriate
- [ ] `as` for type assertions (not angle brackets)
- [ ] `const` assertions for literal types

### Language
- [ ] `const` by default, `let` for reassignment, no `var`
- [ ] One declaration per variable
- [ ] No `Array()` constructor — use `[]`
- [ ] No `for...in` — use `for...of`
- [ ] Arrow functions for callbacks
- [ ] Template strings over concatenation
- [ ] `===` / `!==` (not `==` / `!=`)
- [ ] Object/array literals over constructors
- [ ] No container classes (static-only classes)

---

Full source: `G:\styleguide\tsguide.html`
