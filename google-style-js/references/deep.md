# Google JavaScript Style Guide — Deep Reference

> **Deprecation notice:** This guide is no longer being updated. Google recommends migrating to TypeScript.

---

## 1. Source file basics

### 1.1 File names

File names must be all lowercase and may include underscores (`_`) or dashes (`-`). No additional punctuation.

```
mycontroller.js
my-controller_test.js
data_handler.js
```

Files must be encoded in **UTF-8**.

### 1.2 Special characters

Use the actual Unicode character whenever possible. Escape sequences (`\uXXXX`) are acceptable only when the character is not representable in the source encoding or when escaping is required for readability (e.g., `\n`, `\t`).

```js
// Good — actual Unicode character
const copyright = '©';

// Acceptable — escaped
const lineBreak = '\n';
```

Non-ASCII characters must never be used in string literals for right-to-left text without explicit bidirectional control characters.

---

## 2. Source file structure

### 2.1 Module systems

Google JavaScript code uses one of two module systems:

- **`goog.module`** — Closure-library-based modules
- **ES modules** — Standard ECMAScript modules with `import` / `export`

Both are acceptable. A file must not mix the two.

### 2.2 File section order

Every source file must follow this section order:

1. License or copyright header (if applicable)
2. `@fileoverview` JSDoc (optional but recommended)
3. `goog.module` statement or ES `export` / `import` statements
4. Implementation code

Blank lines separate sections.

### 2.3 `goog.module` — full pattern

```js
/**
 * @fileoverview Description of the file contents.
 */

goog.module('my.app.Car');

const Engine = goog.require('my.app.Engine');
const FuelTank = goog.requireType('my.app.FuelTank');

class Car {
  constructor() {
    /** @private @const {!Engine} */
    this.engine_ = new Engine();
  }

  /**
   * @param {number} liters
   * @return {boolean}
   */
  refuel(liters) {
    return true;
  }
}

exports = Car;
```

### 2.4 `goog.module.declareLegacyNamespace`

Legacy code that must be accessible outside `goog.module` may use:

```js
goog.module('my.app.Car');
goog.module.declareLegacyNamespace();
```

This pattern is discouraged for new code.

### 2.5 Export from `goog.module`

A single value is exported via `exports =`:

```js
exports = Car;
```

Multiple values are exported via an object literal:

```js
exports = {Car, Engine};
```

### 2.6 ES module imports

```js
import Engine from './engine.js';
import {FuelTank} from './fuel_tank.js';
```

### 2.7 ES module exports

```js
export class Car { ... }

// Named exports only — default exports are banned.
```

### 2.8 Named vs default exports

**Named exports are required.** Default exports are forbidden.

```js
// BAD — default export
export default class Car { }

// GOOD — named export
export class Car { }
```

### 2.9 `goog.require` vs `goog.requireType`

Use `goog.require` for values used at runtime:

```js
const Engine = goog.require('my.app.Engine');
```

Use `goog.requireType` for types used only in JSDoc annotations:

```js
/** @type {?my.app.FuelTank} */
this.tank_ = null;

const FuelTank = goog.requireType('my.app.FuelTank');
```

### 2.10 Module naming hierarchy

Module names are `lowerCamelCase` and follow the filesystem path:

```
// file: engine/electric_motor.js
goog.module('engine.electricMotor');
```

---

## 3. Formatting deep dive

### 3.1 Brace style — K&R (1TBS)

Opening brace goes on the same line as the statement, preceded by a space.

```js
// GOOD — K&R style
class Car {
  constructor() {
    if (engine) {
      this.start();
    } else {
      this.stop();
    }
  }
}

// BAD — Allman style
class Car
{
  constructor()
  {
    if (engine)
    {
      this.start();
    }
  }
}
```

### 3.2 Block indentation

Each new block adds +2 spaces of indentation.

```js
for (let i = 0; i < 10; i++) {
  if (condition) {
    doSomething();
  }
}
```

### 3.3 One statement per line

```js
// GOOD
const a = 1;
const b = 2;

// BAD
const a = 1; const b = 2;
```

### 3.4 Column limit: 80 characters

No line may exceed 80 characters. The only exceptions:
- Lines where obeying the limit would break a string literal (use string concatenation instead)
- `goog.module` statements
- URL and `goog.require` lines

### 3.5 Line-wrapping

When a line exceeds 80 characters, break it at a **higher syntactic level**.

#### Operator-first style

When breaking an expression, the operator goes on the **next line** (operator-first / "break before operator"):

```js
// GOOD — operator-first
const result = someLongExpression
    + anotherLongExpression
    + yetAnotherExpression;

// BAD — operator after
const result = someLongExpression +
    anotherLongExpression +
    yetAnotherExpression;
```

#### Continuation indent: +4

Continuation lines are indented **+4 spaces** from the start of the expression:

```js
doSomething(
    argument1,
    argument2,
    argument3);

const value = some.really.long.expression
    && another.expression;
```

#### Where to break

Break after `(` or before a non-closing-paren token:

```js
// GOOD — break after (
someFunction(
    argument1, argument2,
    argument3);

// GOOD — break at operators
const result = condition1 && condition2
    && condition3;
```

### 3.6 Blank lines

- One blank line between top-level blocks (classes, functions, constant definitions)
- One blank line between methods in a class
- One blank line between sections (imports → implementation)
- Use blank lines sparingly inside methods to group logical steps

### 3.7 Horizontal whitespace

#### Required whitespace

Separate reserved words from following `(` or `{`:

```js
// GOOD
if (condition) {
while (true) {
for (let i = 0; i < n; i++) {
switch (x) {
```

Separate `}` from `else`, `catch`, `finally`, `while`:

```js
// GOOD
} else {
} catch (e) {
} finally {
} while (condition);
```

Separate opening `{` from preceding token:

```js
// GOOD
class Car {
if (x) {
return {a: 1};
```

Separate binary/ternary operators from operands:

```js
// GOOD
a + b
c ? 1 : 2
typeof x
```

After commas, semicolons, and type casts:

```js
// GOOD
[1, 2, 3]
for (let i = 0; i < n; i++)
const x = /** @type {number} */ (y);
```

Around colons in object literals and `case`:

```js
// GOOD
{key: value}
case 1:
```

Before opening `(` in control flow statements:

```js
// GOOD
if (x)
while (x)
for (x)
switch (x)
catch (x)
```

#### Forbidden whitespace

No space inside parentheses, brackets, or angle brackets:

```js
// GOOD
foo(x, y)
[1, 2, 3]
{FOO: 1}
foo<a, b>(x)

// BAD
foo( x, y )
[ 1, 2, 3 ]
{ FOO: 1 }
foo< a, b >( x )
```

No space before `)` or after `(`:

```js
// GOOD
if (x)

// BAD
if ( x )
```

No space before commas or semicolons:

```js
// GOOD
[1, 2, 3]
for (let i = 0; i < n; i++)

// BAD
[1 , 2 , 3]
for (let i = 0 ; i < n ; i++)
```

No space between a function name and `(`:

```js
// GOOD
foo()
foo(1)

// BAD
foo ()
foo (1)
```

No space between `[` of array access and `]`:

```js
// GOOD
arr[0]

// BAD
arr[ 0 ]
```

No space inside string template `${}`:

```js
// GOOD
`Hello, ${name}!`

// BAD
`Hello, ${ name }!`
```

### 3.8 Horizontal alignment not required

Aligning variable names or values across lines is permitted but not required.

```js
// Allowed but not required
const a        = 1;
const lengthy  = 2;
const longer__ = 3;

// Also fine
const a = 1;
const lengthy = 2;
const longer__ = 3;
```

### 3.9 Grouping parentheses

Use parentheses only when they are needed or improve clarity. Do not add redundant parentheses around an entire expression.

```js
// GOOD
const result = (a * b) + (c * d);

// UNNECESSARY
const result = (a + b);

// GOOD — clarifies intent
if ((x || y) && z) { ... }
```

### 3.10 Specific constructs

#### Arrays

Use square brackets — no `new Array()` except for size:

```js
// GOOD
const arr = [1, 2, 3];

// BAD
const arr = new Array(1, 2, 3);

// ACCEPTABLE (single argument = length)
const arr = new Array(10);
```

No trailing commas in array literals:

```js
// GOOD
const arr = [1, 2, 3];

// BAD
const arr = [1, 2, 3,];
```

No sparse arrays:

```js
// BAD — sparse
const arr = [1, , 2];
```

#### Objects

Use curly braces. Quote keys only when needed (reserved words, special characters):

```js
// GOOD
const obj = {key: 'value', 'for': 1};

// BAD — unnecessary quoting
const obj = {'key': 'value'};
```

Trailing comma on last property is forbidden:

```js
// GOOD
const obj = {a: 1, b: 2};

// BAD
const obj = {a: 1, b: 2,};
```

Computed property names are allowed:

```js
const obj = {[key]: value};
```

Shorthand properties and methods are allowed:

```js
const obj = {a, b, method() {}};
```

#### Classes

```js
class Car extends Vehicle {
  constructor(make, model) {
    super(make);
    this.model_ = model;
  }

  start() {
    this.engine_.ignite();
  }
}
```

Class methods use `methodName() {}` syntax, not property-assigned functions.

No `@constructor` JSDoc needed — the `class` keyword is sufficient.

#### Functions

Function declarations vs arrow functions — see Language rules section.

Functions should not have a space before `(`:

```js
// GOOD
function foo() {}

// BAD
function foo () {}
```

#### String literals

Use single quotes (`'`) consistently, or template literals when interpolation or multi-line is needed:

```js
// GOOD
const name = 'John';
const greeting = `Hello, ${name}!`;
const multiLine = `Line one
Line two`;

// BAD — double quotes
const name = "John";
```

Long strings that exceed 80 characters should be concatenated:

```js
const message = 'This is a very long string that exceeds the 80 character '
    + 'limit and must be broken across multiple lines.';
```

#### Number literals

No leading zeros (except `0x` for hex). Use `0.5` not `.5`. Use `1e3` not `1000` only when appropriate.

```js
// GOOD
const x = 0.5;
const hex = 0xFF;

// BAD
const x = .5;
const y = 01;
```

#### Control structures

All control structures must use braces for multi-line blocks. Braces are optional only for single-line bodies:

```js
// GOOD
if (condition) {
  doSomething();
  doSomethingElse();
}

// ACCEPTABLE — single-line, no braces
if (condition) doSomething();

// BAD — multi-line without braces
if (condition)
  doSomething();
  doSomethingElse();
```

The `else` keyword goes on the same line as the closing `}`:

```js
if (x) {
  handleX();
} else if (y) {
  handleY();
} else {
  handleDefault();
}
```

---

## 4. Naming deep dive

### 4.1 Rules common to all identifiers

Identifiers must use only:
- Uppercase letters `A–Z`
- Lowercase letters `a–z`
- Digits `0–9`
- Underscore `_`
- Dollar sign `$` (only for machine-generated code)

```js
// GOOD
let count = 0;
const MAX_SIZE = 100;
let _privateHelper;

// BAD
let こんにちは = 'hello';  // no Unicode letters
let my-var = 1;             // no hyphens
```

### 4.2 Rules by identifier type

#### Packages

`lowerCamelCase`

```js
goog.module('myApp.models.car');
```

Module names mirror the directory structure and file name:

```
src/my_app/models/car.js
→ goog.module('myApp.models.car');
```

#### Classes, interfaces, enums, types

`UpperCamelCase`

```js
class CarEngine {}
interface Drivable {}
enum Color { RED, GREEN, BLUE }
```

#### Methods

`lowerCamelCase`. Private methods may (but are not required to) end with a trailing underscore `_`.

```js
class Car {
  startEngine() { ... }
  _calculateSpeed_() { ... }
}
```

#### Constants

`CONSTANT_CASE` — all uppercase with underscores separating words.

A value is a constant only when it is **deeply immutable**:
- The value is assigned once
- The value is `const`
- If it is an array, no elements are ever mutated
- If it is an object, none of its properties are ever mutated
- No external mutations are observable

```js
// GOOD — primitive constant
const MAX_RETRIES = 3;

// GOOD — deeply immutable frozen object
const CONFIG = Object.freeze({url: 'https://example.com', retries: 3});

// BAD — object is mutated
const config = {url: 'https://example.com'};
config.url = 'https://other.com';

// BAD — array is mutated
const NUMBERS = [1, 2, 3];
NUMBERS.push(4);
```

#### Non-constant fields

`lowerCamelCase`. Private fields may end with a trailing underscore.

```js
class Car {
  constructor() {
    this.engineType_ = 'V8';
    this.color = 'red';
  }
}
```

#### Parameters

`lowerCamelCase`

```js
function connect(url, timeoutMs) { ... }
```

#### Local variables

`lowerCamelCase`

```js
function calculate() {
  let totalCount = 0;
  const multiplier = 2;
}
```

#### Template parameters

`ALL_CAPS` — a single word in all caps, or multiple words separated by underscores. This distinguishes them from class names.

```js
/**
 * @template T
 * @param {!Array<T>} arr
 * @return {T}
 */
function first(arr) {
  return arr[0];
}

/**
 * @template KEY_T, VALUE_T
 */
class MapWrapper {}
```

#### Module-local names

Internal names that are not exported follow the same rules as local variables.

```js
goog.module('my.app.CarFactory');

const DEFAULT_COLOR = 'black';   // module-level constant
let internalCounter = 0;         // module-level mutable

class CarFactory { ... }

exports = CarFactory;
```

### 4.3 Camel case defined

Convert the phrase to ASCII, remove apostrophes, then split into words (spaces and remaining punctuation are separators). Apply the following:

- **UpperCamelCase**: Capitalize the first letter of each word, including the first.
- **lowerCamelCase**: Capitalize the first letter of each word *except* the first, which is lowercase.
- **Acronyms and initialisms**: Each acronym is treated as a single word.

Examples:

| Phrase | UpperCamelCase | lowerCamelCase |
|---|---|---|
| XML HTTP request | `XmlHttpRequest` | `xmlHttpRequest` |
| new customer ID | `NewCustomerId` | `newCustomerId` |
| inner HTML | `InnerHtml` | `innerHtml` |
| JSON parser | `JsonParser` | `jsonParser` |

Note: `XmlHttpRequest`, not `XMLHTTPRequest`; `loadJsonUrl`, not `loadJSONUrl`.

---

## 5. JSDoc deep dive

### 5.1 General form

JSDoc blocks begin with `/**` and end with `*/`. Every line (except the first and last) begins with ` * `.

```js
/**
 * A brief description of the function.
 * @param {string} name The name of the thing.
 * @return {number} The computed value.
 */
function compute(name) {
  return name.length;
}
```

### 5.2 Markdown in JSDoc

JSDoc supports Markdown for formatting descriptions:

```js
/**
 * Calculates the total.
 *
 * Steps:
 * 1. Validate input
 * 2. Compute result
 * 3. Return value
 *
 * See {@link #otherMethod}.
 *
 * @param {Array<number>} values
 * @return {number}
 */
function sum(values) { ... }
```

### 5.3 JSDoc tags — complete list

| Tag | Description |
|---|---|
| `@param {type} name desc` | Parameter documentation |
| `@return {type} desc` / `@returns {type} desc` | Return value |
| `@type {type}` | Inline type annotation |
| `@private` | Visibility — not part of public API |
| `@protected` | Visibility — accessible to subclasses |
| `@public` | Visibility — part of public API |
| `@const` | Marks a constant |
| `@final` | Method or class cannot be overridden / extended |
| `@export` | Marks a symbol for the compiler to export |
| `@template T` | Declares a generic type parameter |
| `@implements {Interface}` | Class implements an interface |
| `@override` | Method overrides a superclass method |
| `@extends {SuperClass}` | Class extends another class |
| `@fileoverview` | File-level description |
| `@author` | Author (optional, not recommended) |
| `@see` | Reference to related code |
| `@throws {Type}` | Exception that may be thrown |
| `@deprecated` | Marks a symbol as deprecated |
| `@suppress {warning}` | Suppresses a Closure Compiler warning |
| `@dict` | Marks an object as a dict (no prototype) |
| `@struct` | Marks an object as a struct (no new properties) |
| `@nocollapse` | Prevents property collapsing by compiler |
| `@enum {Type}` | Defines an enum type |
| `@typedef {Type}` | Defines a type alias |
| `@return {!Promise<void>}` | Async return type |

### 5.4 Line wrapping in JSDoc

Block descriptions: wrap at 80 characters. Parameter descriptions should be aligned if wrapping:

```js
/**
 * @param {string} veryLongParameterName Description that goes on too long
 *     and must wrap to the next line with an extra 4-space indent.
 * @param {number} anotherParam Shorter description.
 */
```

### 5.5 Top / file-level comments

```js
/**
 * @fileoverview Description of what this file does and how it
 *     fits into the overall application.
 */
```

### 5.6 Class comments

Every class must have a JSDoc if its purpose is non-obvious:

```js
/**
 * A car with an electric engine.
 * @implements {Drivable}
 */
class ElectricCar extends Vehicle implements Drivable {
  ...
}
```

### 5.7 Method / function comments

```js
/**
 * Starts the engine.
 * @param {number} rpm Target revolutions per minute.
 * @param {boolean=} silent If true, suppress startup sound.
 * @return {boolean} True if the engine started successfully.
 * @throws {EngineError} If the engine fails to start.
 */
start(rpm, silent) {
  ...
}
```

### 5.8 Property comments

```js
class Car {
  constructor() {
    /** @type {string} */
    this.model = '';

    /** @private @const {!Engine} */
    this.engine_ = new Engine();
  }
}
```

### 5.9 Type annotations overview

| Pattern | Meaning |
|---|---|
| `{string}` | Nullable string |
| `{!string}` | Non-null string |
| `{?string}` | Nullable string (explicit) |
| `{number}` | Nullable number |
| `{boolean}` | Nullable boolean |
| `{?}` | Unknown type |
| `{!Object}` | Non-null object |
| `{!Array<number>}` | Non-null array of numbers |
| `{!Object<string, number>}` | String-keyed map of numbers |
| `{function(string): number}` | Function type |
| `{undefined}` | Undefined |
| `{void}` | Return nothing |
| `{!Promise<!Array<string>>}` | Promise of string array |

`@type`, `@param`, and `@return` use this type syntax.

### 5.10 Visibility annotations

```js
/**
 * @private
 */
this.internalState_ = 0;

/**
 * @protected
 */
this.protectedValue = 0;

/**
 * @public
 */
this.publicValue = 0;
```

### 5.11 `@const` and `@final`

`@const` marks a variable as constant — the compiler may inline it.

```js
/** @const {string} */
const APP_NAME = 'MyApp';
```

`@final` prevents a method from being overridden or a class from being extended:

```js
/**
 * @final
 */
class ImmutableBox { ... }
```

### 5.12 `@export`

Use `@export` to make a symbol visible outside compiled code:

```js
/**
 * @export
 */
class PublicApi { ... }
```

---

## 6. Language rules deep dive

### 6.1 Local variable declarations — `const` / `let`

Use `const` by default. Use `let` only when the variable **must** be reassigned. `var` is forbidden.

```js
// GOOD
const maxCount = 100;
let counter = 0;
counter++;

// BAD — var
var total = 0;

// BAD — unnecessary let
let maxCount = 100;
```

One declaration per variable:

```js
// GOOD
const a = 1;
const b = 2;

// BAD
const a = 1, b = 2;
```

### 6.2 Array literals

Use `[]` instead of `new Array()`:

```js
// GOOD
const arr = [1, 2, 3];

// BAD
const arr = new Array(1, 2, 3);
```

`new Array(length)` is allowed only when the single numeric argument is intended to set the length:

```js
const arr = new Array(10);  // arr.length === 10, all elements undefined
```

No sparse arrays:

```js
// BAD
const arr = [1, , 2];
```

### 6.3 Object literals

Use `{}` instead of `new Object()`:

```js
// GOOD
const obj = {a: 1};

// BAD
const obj = new Object();
```

Do not quote keys unless they contain special characters:

```js
// GOOD
const obj = {key: 'value', 'class': 1};

// BAD
const obj = {'key': 'value'};
```

No trailing commas:

```js
// BAD
const obj = {a: 1, b: 2,};
```

### 6.4 String literals

Prefer single quotes `'`. Template literals (backticks) are preferred when:

- The string contains interpolation: `` `${name}` ``
- The string spans multiple lines

```js
// GOOD
const name = 'John';
const greeting = `Hello, ${name}!`;

// ACCEPTABLE but less common
const name = "John";
```

For line-wrapping long strings, concatenate with `+`:

```js
const message = 'This is a very long string that exceeds the 80 character '
    + 'limit and must be broken across multiple lines.';
```

### 6.5 Number literals

Use `0x` for hexadecimals, `0o` for octals, `0b` for binaries. No leading zeros:

```js
// GOOD
const x = 0.5;
const hex = 0xFF;
const oct = 0o77;
const bin = 0b1010;

// BAD
const x = .5;         // leading dot
const y = 1.;         // trailing dot
const z = 01;         // legacy octal
```

### 6.6 Control structures — `for-in` and `for-of`

Use `for-of` for iterating over iterables:

```js
for (const item of items) {
  process(item);
}
```

`for-in` is allowed only for iterating over object keys (not arrays), and must check `hasOwnProperty`:

```js
for (const key in object) {
  if (Object.prototype.hasOwnProperty.call(object, key)) {
    console.log(key, object[key]);
  }
}
```

### 6.7 Exceptions

Exceptions are an important part of the language. Throw `Error` or a subclass — never throw primitives:

```js
// GOOD
throw new Error('Something went wrong');
throw new RangeError('Out of bounds');

// BAD
throw 'Something went wrong';
throw 42;
```

Custom error types are allowed:

```js
class EngineError extends Error {
  constructor(message) {
    super(message);
    this.name = 'EngineError';
  }
}
```

### 6.8 Functions

#### Function declarations vs expressions

Prefer function declarations for named functions:

```js
// GOOD — declaration
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}
```

Assigning a function expression to a `const` is acceptable when the function must be reassignable or passed as a callback:

```js
const handler = function(event) {
  event.preventDefault();
};
```

#### IIFEs

Immediately-invoked function expressions are allowed but rarely needed with ES modules:

```js
const value = (function() {
  return computeExpensiveValue();
})();
```

### 6.9 Arrow functions

Arrow functions are preferred over `function` expressions for callbacks, especially when preserving lexical `this`:

```js
// GOOD
items.map(item => item.value);
items.forEach((item, index) => {
  process(item, index);
});

// AVOID when not needed
items.map(function(item) { return item.value; });
```

Use parentheses around a single parameter only when the parameter has a type annotation or default value:

```js
// GOOD
items.map(item => item.value);
items.map((item) => item.value);   // also acceptable

// Parentheses required with defaults or destructuring
items.map((item, index) => item + index);
items.map(({id, name}) => ({id, name}));
```

### 6.10 Classes

Use ES6 `class` syntax. Avoid `prototype` manipulation:

```js
// GOOD
class Car {
  constructor(make) {
    this.make_ = make;
  }

  start() { ... }
}

// BAD
function Car(make) {
  this.make_ = make;
}
Car.prototype.start = function() { ... };
```

### 6.11 `this`

`this` should only be used in class constructors, methods, and arrow functions that close over `this`. Avoid capturing `this` with `const self = this;` — use arrow functions instead:

```js
// GOOD — arrow function captures lexical this
class Car {
  startAsync() {
    setTimeout(() => {
      this.running_ = true;   // arrow function captures outer this
    }, 0);
  }
}

// AVOID
class Car {
  startAsync() {
    const self = this;
    setTimeout(function() {
      self.running_ = true;
    }, 0);
  }
}
```

### 6.12 Equality checks

Always use `===` and `!==`. Never use `==` or `!=`:

```js
// GOOD
if (x === 0) { ... }
if (x !== null) { ... }

// BAD
if (x == 0) { ... }
if (x != null) { ... }
```

### 6.13 Type coercion avoidance

Do not rely on implicit type coercion. Be explicit:

```js
// GOOD
const str = String(value);
const num = Number(value);
const bool = Boolean(value);

// BAD
const str = '' + value;       // implicit string coercion
const num = +value;           // implicit numeric coercion
const bool = !!value;         // implicit boolean coercion
```

### 6.14 `switch` statements

`switch` is allowed but must include a `default` case:

```js
switch (status) {
  case STATUS_ACTIVE:
    handleActive();
    break;
  case STATUS_INACTIVE:
    handleInactive();
    break;
  default:
    handleUnknown();
}
```

Fall-through is allowed only with a comment indicating intentional fall-through:

```js
switch (x) {
  case 1:
  case 2:   // intentional fall-through
    handleOneOrTwo();
    break;
  case 3:
    handleThree();
    break;
  default:
    handleDefault();
}
```

### 6.15 `with` / `eval` / `Function` constructor

- `with` is forbidden.
- `eval` is forbidden (use `JSON.parse` for parsing JSON).
- The `Function` constructor is forbidden.

```js
// BAD — all of these are prohibited
with (obj) { x = y; }
const result = eval('2 + 2');
const fn = new Function('a', 'b', 'return a + b');
```

---

## 7. Review checklist

### Naming

- [ ] Packages use `lowerCamelCase`
- [ ] Classes / interfaces / types use `UpperCamelCase`
- [ ] Methods use `lowerCamelCase`
- [ ] Constants (deeply immutable) use `CONSTANT_CASE`
- [ ] Non-constant fields use `lowerCamelCase`
- [ ] Parameters and local variables use `lowerCamelCase`
- [ ] Template parameters use `ALL_CAPS`
- [ ] Acronyms are treated as whole words (`XmlHttpRequest`, not `XMLHTTPRequest`)

### Formatting

- [ ] No line exceeds 80 characters
- [ ] 2-space indentation, no tabs
- [ ] K&R brace style (1TBS)
- [ ] One statement per line
- [ ] Spaces inside non-empty block braces
- [ ] No spaces inside parentheses
- [ ] Continuation lines indented +4
- [ ] Operator-first style for wrapped expressions
- [ ] Blank lines between top-level blocks

### JSDoc

- [ ] `@fileoverview` at top of file (if useful)
- [ ] `@param` and `@return` on all public methods
- [ ] `@private` / `@protected` / `@public` as appropriate
- [ ] `@const` for constants
- [ ] `@template` for generic types
- [ ] `@override` where applicable

### Language

- [ ] `const` / `let` only — no `var`
- [ ] One declaration per variable
- [ ] Array and object literals over constructors
- [ ] No sparse arrays, no trailing commas
- [ ] Single quotes for strings (or template literals)
- [ ] Arrow functions for callbacks
- [ ] `===` / `!==` — no `==` / `!=`
- [ ] No `with`, `eval`, or `Function` constructor
- [ ] Explicit type coercion
- [ ] Default case in `switch` statements
- [ ] Braces for all multi-line blocks

### Modules

- [ ] Consistent module system (`goog.module` or ES)
- [ ] No default exports — named exports only
- [ ] Correct `goog.require` / `goog.requireType` usage
- [ ] File sections in order: license → `@fileoverview` → module → implementation

---

Full source: `G:\styleguide\jsguide.html`
