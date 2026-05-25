# Google Objective-C Style Guide — Deep Reference

---

## 1. Principles

### Optimize for the Reader

Code is read far more often than it is written. The style guide exists to make code easier to understand, navigate, and maintain. Every decision — naming, formatting, type choice — should default toward clarity for an unfamiliar reader.

### Be Consistent

When editing existing code, match the surrounding style even if it diverges from the guide. Consistency within a file or module matters more than strict adherence to a rule. When contributing to a new project, adopt its existing conventions.

### Be Consistent with Apple SDKs

Objective-C is inextricably tied to Apple's frameworks. Follow Apple's naming and design patterns (Cocoa naming conventions, memory management model) unless the guide explicitly overrides them. This reduces surprise for developers familiar with the platform.

### Style Rules Should Pull Their Weight

Every rule must justify its existence. If a rule is burdensome and provides minimal benefit, it should be removed. Rules exist to prevent real bugs, improve readability at scale, or enforce consistency across a large team.

---

## 2. Example

### Header File (`GTMBook.h`)

```objectivec
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@protocol GTMBookDelegate;

typedef NS_ENUM(NSInteger, GTMBookType) {
  GTMBookTypePDF,
  GTMBookTypeEPUB,
  GTMBookTypeAudio,
};

extern NSString *const GTMBookErrorDomain;

@interface GTMBook : NSObject <NSCoding>

@property(nonatomic, copy) NSString *title;
@property(nonatomic, copy, nullable) NSString *author;
@property(nonatomic, readonly) GTMBookType type;
@property(nonatomic, weak, nullable) id<GTMBookDelegate> delegate;

- (instancetype)initWithTitle:(NSString *)title
                         type:(GTMBookType)type NS_DESIGNATED_INITIALIZER;

- (BOOL)open;
- (void)close;

@end

@protocol GTMBookDelegate <NSObject>
@optional
- (void)bookDidOpen:(GTMBook *)book;
- (void)bookDidClose:(GTMBook *)book;
@end

NS_ASSUME_NONNULL_END
```

### Implementation File (`GTMBook.m`)

```objectivec
#import "GTMBook.h"

NSString *const GTMBookErrorDomain = @"GTMBookErrorDomain";

static NSString *const kDefaultAuthor = @"Unknown";

@interface GTMBook () {
  NSInteger _currentPage;
}

@property(nonatomic, assign) NSInteger currentPage;

@end

@implementation GTMBook

#pragma mark - Initialization

- (instancetype)initWithTitle:(NSString *)title
                         type:(GTMBookType)type {
  self = [super init];
  if (self) {
    _title = [title copy];
    _type = type;
    _currentPage = 0;
  }
  return self;
}

- (instancetype)init {
  return [self initWithTitle:@"" type:GTMBookTypePDF];
}

#pragma mark - Public Methods

- (BOOL)open {
  if ([self.delegate respondsToSelector:@selector(bookDidOpen:)]) {
    [self.delegate bookDidOpen:self];
  }
  return YES;
}

- (void)close {
  _currentPage = 0;
  if ([self.delegate respondsToSelector:@selector(bookDidClose:)]) {
    [self.delegate bookDidClose:self];
  }
}

#pragma mark - NSCoding

- (void)encodeWithCoder:(NSCoder *)coder {
  [coder encodeObject:_title forKey:@"title"];
  [coder encodeInteger:_type forKey:@"type"];
}

- (nullable instancetype)initWithCoder:(NSCoder *)coder {
  self = [super init];
  if (self) {
    _title = [[coder decodeObjectForKey:@"title"] copy];
    _type = [coder decodeIntegerForKey:@"type"];
  }
  return self;
}

@end
```

---

## 3. Naming Deep Dive

### Descriptive Names

Every name must be descriptive and unambiguous. Avoid abbreviations unless they are universally understood (e.g., `URL`, `ID`, `TIFF`, `EXIF`). Single-letter names are never acceptable outside trivial loop counters.

### Inclusive Language

Use inclusive terminology. Avoid terms like "blacklist" / "whitelist" (prefer "allowlist" / "denylist"), "master" / "slave" (prefer "primary" / "secondary"), and other non-inclusive language in identifiers, comments, and documentation.

### File Names

File names must reflect the primary class they contain. Case sensitivity matters on the filesystem; match the class name exactly.

| Extension | Purpose |
|---|---|
| `.h` | Interface / header |
| `.m` | Objective-C implementation |
| `.mm` | Objective-C++ implementation |

Categories and extensions should use `ClassName+CategoryName.h` / `.m` convention: `GTMBook+GTMBookStorage.h`.

### Prefixes

Every class, protocol, category, global function, and global constant must have a 3+ character prefix (e.g., `GTM`, `ABC`). Prefixes prevent symbol collisions across libraries and modules. Two-letter prefixes are reserved by Apple.

- Classes: `GTMBook`
- Protocols: `GTMBookDelegate`
- Categories: `GTMBook (GTMBookAdditions)`
- Global constants: `GTMBookErrorDomain`
- Global functions: `GTMBookValidate()`

### Class Names

UpperCamelCase with prefix. Nouns or noun phrases. Example: `GTMHttpRequest`, `GTMBookParser`.

### Category Naming

Categories extend existing classes. Name format: `ClassName (CategoryName)`. Methods added by a category must use a lowercase prefix (`gtm_`) to avoid collisions:

```objectivec
@interface GTMBook (GTMBookStorage)
- (BOOL)gtm_saveToPath:(NSString *)path;
@end
```

### Objective-C Method Names

Methods should read like sentences, forming a clear action-receiver pair. Use lowerCamelCase.

```objectivec
- (void)insertObject:(id)obj atIndex:(NSUInteger)index;     // Good
- (void)insert:(id)obj at:(NSUInteger)index;                // Bad — loses meaning
- (void)setBackgroundColor:(UIColor *)color;                 // Good
- (void)setBackground:(UIColor *)color;                      // Bad — too vague
```

Parameter labels should describe the argument:

```objectivec
- (id)initWithNibName:(NSString *)nibName bundle:(NSBundle *)bundle;  // Good
- (id)initWithNib:(NSString *)nib bundle:(NSBundle *)bundle;          // Bad
```

### Function Names

Non-static functions use PascalCase with the project prefix: `GTMBookOpenFile()`. Static (file-local) functions may omit the prefix but should still use PascalCase: `static void BookOpenFile()`. Functions perform actions; prefer verb-noun: `GTMBookValidateMetadata()`.

### Variable Names

| Scope | Convention | Example |
|---|---|---|
| Local | lowerCamelCase | `currentItem`, `isValid` |
| Instance variable | Leading underscore | `_ivarName` |
| Global | `g` prefix + PascalCase | `gSharedCache`, `gLoggingEnabled` |
| Static (file) | lowerCamelCase | `static id sSharedInstance` |

Instance variables should generally be private and accessed via properties. Declare ivars in the `@implementation` or class extension, not in the public `@interface`.

### Constants

Use `const` over `#define`. Public constants use the project prefix; file-static constants use `k` prefix.

```objectivec
// Public — project prefix
extern NSString *const GTMBookErrorDomain;

// File-static — k prefix
static const NSTimeInterval kAnimationDuration = 0.3;

// File-static integer
static const NSInteger kMaxRetryCount = 3;
```

### Enum Naming

Use `NS_ENUM` macro. Enum values are 3rd-person: `GTMBookTypePDF`. The enum type name matches the typedef name.

```objectivec
typedef NS_ENUM(NSInteger, GTMBookType) {
  GTMBookTypePDF,
  GTMBookTypeEPUB,
  GTMBookTypeAudio,
};
```

Use `NS_OPTIONS` for bitmask flags:

```objectivec
typedef NS_OPTIONS(NSUInteger, GTMBookOptions) {
  GTMBookOptionsNone      = 0,
  GTMBookOptionsBold      = 1 << 0,
  GTMBookOptionsItalic    = 1 << 1,
  GTMBookOptionsUnderline = 1 << 2,
};
```

---

## 4. Types and Declarations Deep Dive

### Method Declarations in `@interface`

Methods in the public `@interface` should be grouped logically. Order declarations by their access level and purpose:

1. Properties
2. Initializers (designated first)
3. Public methods
4. Protocol conformance methods (in a class extension if private)

### Local Variables

Declare variables in the narrowest scope possible, preferably at the point of first use. Initialize at declaration.

```objectivec
// Good — declared at point of use
NSInteger itemCount = [self itemCount];
for (NSInteger i = 0; i < itemCount; i++) { ... }

// Bad — declared far from use
NSInteger itemCount;
// ... 20 lines of unrelated code ...
itemCount = [self itemCount];
```

### Static Variables

Use `static` for file-scoped state instead of globals when the symbol does not need cross-file visibility.

```objectivec
// File-local static variable
static NSString *const kServiceName = @"GTMBookService";
```

### Unsigned Integers

Avoid `unsigned int` for general-purpose arithmetic. Implicit casting between signed and unsigned can produce surprising behavior (e.g., `-1 > 0u` is `true`). Use `NSInteger` / `NSUInteger` or exact-width types.

### Type Sizes and 32/64-bit Safety

Types with inconsistent sizes across architectures:

| Type | 32-bit | 64-bit |
|---|---|---|
| `int` / `unsigned int` | 32 | 32 |
| `long` / `unsigned long` | 32 | 64 |
| `NSInteger` / `NSUInteger` | 32 | 64 |
| `CGFloat` | 32 | 64 |

When reading/writing to disk or serialization, use exact-width types (`int32_t`, `int64_t`). When interacting with Cocoa APIs, use `NSInteger` / `NSUInteger` / `CGFloat`.

---

## 5. Formatting Deep Dive

### Line Length

Maximum 100 characters per line. Break long lines at a logical boundary (operator, comma, colon). Indent continuation lines by 4 spaces.

### Indentation

2 spaces. No tabs. Set your editor to insert spaces when the Tab key is pressed.

### Braced Blocks — K&R Style

Opening brace on the same line as the statement, closing brace on its own line:

```objectivec
if (condition) {
  // body
} else {
  // other body
}

for (NSInteger i = 0; i < count; i++) {
  // body
}

while (condition) {
  // body
}
```

### `@interface` / `@implementation` Formatting

Opening brace on the same line as `@interface` / `@implementation`:

```objectivec
@interface GTMBook : NSObject {
 @private
  NSString *_title;
}
@end

@implementation GTMBook
@end
```

### `@public` / `@private` / `@protected`

Indented 2 spaces from the `@interface` line. The colon after the access specifier is followed by ivar declarations:

```objectivec
@interface GTMBook : NSObject {
 @public
  NSString *publicTitle;
 @private
  NSString *_privateTitle;
}
```

### Protocols

Protocol conformance on the `@interface` line. Protocol methods in the protocol block should follow the same method formatting rules.

```objectivec
@interface GTMBook : NSObject <NSCoding, GTMBookDelegate>
```

### Method Declarations

```objectivec
- (void)doSomethingWithParam:(NSString *)param;
- (nullable instancetype)initWithTitle:(NSString *)title
                                 style:(GTMBookType)style
                               options:(GTMBookOptions)options;
```

Align colons when a method spans multiple lines. The first line is indented normally; continuation lines align the colons with the first line's colon.

### Method Calls

Same as declarations — colon alignment across continuation lines:

```objectivec
[self doSomethingWithParam:paramValue
                     style:styleValue
                   options:optionsValue];
```

Parentheses hugging the last argument:

```objectivec
// Good
[self doSomethingWithParam:paramValue
                     style:styleValue];

// Bad — hanging parentheses
[self doSomethingWithParam:paramValue
                     style:styleValue
];
```

### Function Declarations

```objectivec
NSInteger GTMBookCalculatePageCount(NSString *text, CGFloat fontSize);

static NSInteger CalculateWordCount(NSString *text) {
  // ...
}
```

### Blocks / Lambdas

```objectivec
// Inline block
[UIView animateWithDuration:0.3
                 animations:^{
                   self.view.alpha = 0.0;
                 }
                 completion:^(BOOL finished) {
                   [self dismissViewControllerAnimated:NO completion:nil];
                 }];
```

Block as a property:

```objectivec
@property(nonatomic, copy) void (^completionHandler)(BOOL success, NSError *error);
```

### Container Literals

Use Objective-C container literals and subscripting. No trailing comma.

```objectivec
NSArray *items = @[@1, @2, @3];
NSDictionary *mapping = @{
  @"key1" : value1,
  @"key2" : value2,
};
```

Opening brace on same line as `@[` or `@{`. Continuation lines indented 2 spaces. Closing `]` / `}` on its own line for multi-line containers.

### Array / Dictionary Indexing

Use subscript notation:

```objectivec
id item = items[0];
id val = mapping[@"key1"];
```

Not `[items objectAtIndex:0]` or `[mapping objectForKey:@"key1"]`.

---

## 6. Language Features Deep Dive

### ARC

Automatic Reference Counting is required. No manual `retain` / `release` / `autorelease` calls. Do not use `NSAutoReleasePool` — use `@autoreleasepool` blocks.

### `@property` Attributes

Every property must explicitly declare its attributes:

| Attribute | Usage |
|---|---|
| `atomic` / `nonatomic` | `nonatomic` for UI and most properties; `atomic` when thread safety via accessors is required |
| `readwrite` / `readonly` | Explicitly declare |
| `strong` / `weak` / `copy` / `assign` | `copy` for `NSString`, blocks, and value-type objects; `weak` for delegates to avoid retain cycles; `strong` owns; `assign` for C primitives |
| `nullable` / `nonnull` | Use within `NS_ASSUME_NONNULL_BEGIN` / `END` |

```objectivec
@property(nonatomic, copy) NSString *name;
@property(nonatomic, weak, nullable) id<GTMBookDelegate> delegate;
@property(nonatomic, readonly, assign) GTMBookType type;
```

### Ownership Qualifiers

| Qualifier | Description |
|---|---|
| `__strong` | Default. Object is retained. |
| `__weak` | Zeroing weak reference. Automatically nil'd when object deallocates. |
| `__unsafe_unretained` | Non-zeroing weak reference. Unsafe; pointer may dangle. |
| `__autoreleasing` | Used for out-parameters (`NSError **`). |

```objectivec
__weak typeof(self) weakSelf = self;
```

### `NSString` / `NSArray` / `NSDictionary` Literals

Always prefer literals over factory methods:

```objectivec
// Good
NSString *greeting = @"Hello";
NSArray *list = @[item1, item2];
NSDictionary *dict = @{key: value};

// Avoid
NSString *greeting = [NSString stringWithUTF8String:"Hello"];
NSArray *list = [NSArray arrayWithObjects:item1, item2, nil];
NSDictionary *dict = [NSDictionary dictionaryWithObject:value forKey:key];
```

### `#define` Avoidance

Do not use `#define` for constants. Use `const` or `enum`. Use `#define` only for:
- Conditional compilation (`#ifdef DEBUG`)
- Include guards (though `#pragma once` is preferred)
- Macro functions when a function is insufficient

```objectivec
// Good
static const CGFloat kPadding = 16.0;

// Bad
#define kPadding 16.0
```

### Nullability Annotations

Use `_Nullable` / `_Nonnull` or `__nullable` / `__nonnull` in C/ObjC pointer types. Enclose headers in `NS_ASSUME_NONNULL_BEGIN` / `END` to assume nonnull by default.

```objectivec
NS_ASSUME_NONNULL_BEGIN

@interface GTMBook : NSObject
@property(nonatomic, copy, nullable) NSString *author;
- (nullable instancetype)initWithContentsOfURL:(NSURL *)url error:(NSError **)error;
@end

NS_ASSUME_NONNULL_END
```

### Lightweight Generics

Use lightweight generics for type safety with collections:

```objectivec
@property(nonatomic, strong) NSArray<GTMBook *> *books;
@property(nonatomic, strong) NSDictionary<NSString *, GTMBook *> *bookMap;

- (void)addBook:(GTMBook *)book;
- (nullable GTMBook *)bookForIdentifier:(NSString *)identifier;
```

### `NS_ENUM` / `NS_OPTIONS`

Always use these macros. They provide type-checking and switch-case coverage warnings.

```objectivec
typedef NS_ENUM(NSInteger, GTMBookState) {
  GTMBookStateClosed,
  GTMBookStateOpening,
  GTMBookStateOpen,
  GTMBookStateClosing,
};

typedef NS_OPTIONS(NSUInteger, GTMBookFeatures) {
  GTMBookFeaturesNone       = 0,
  GTMBookFeaturesBookmarks  = 1 << 0,
  GTMBookFeaturesAnnotations = 1 << 1,
  GTMBookFeaturesSearch     = 1 << 2,
};
```

### `instancetype`

Use `instancetype` as the return type of `init` methods and class factory methods. This enables subclasses to return the correct type.

```objectivec
- (instancetype)initWithTitle:(NSString *)title;
+ (instancetype)bookWithTitle:(NSString *)title;
```

### `NS_DESIGNATED_INITIALIZER`

Mark designated initializers. All other initializers must delegate to the designated initializer.

```objectivec
- (instancetype)initWithTitle:(NSString *)title
                         type:(GTMBookType)type NS_DESIGNATED_INITIALIZER;

- (instancetype)init {
  return [self initWithTitle:@"" type:GTMBookTypePDF];
}

- (instancetype)initWithTitle:(NSString *)title {
  return [self initWithTitle:title type:GTMBookTypePDF];
}
```

### Class and Instance Methods in Implementation

Group methods by pragma mark. Typically order:
1. `@synthesize` / `@dynamic` (rare — auto-synthesize preferred)
2. `+load`, `+initialize` (avoid if possible)
3. Designated initializer
4. Other initializers
5. `dealloc`
6. Public methods
7. Private methods (class extension)
8. Protocol methods

```objectivec
@implementation GTMBook

#pragma mark - Initialization

- (instancetype)initWithTitle:(NSString *)title
                         type:(GTMBookType)type {
  // ...
}

#pragma mark - Public Methods

- (BOOL)open { ... }

#pragma mark - GTMBookDelegate

- (void)bookDidOpen:(GTMBook *)book { ... }

@end
```

---

## 7. Cocoa Patterns

### Designated Initializers

Every class must have exactly one designated initializer (unless it declares none and inherits from NSObject). The designated initializer calls `[super init...]`. All other initializers call `self`'s designated initializer.

Rules:
- The designated initializer is annotated with `NS_DESIGNATED_INITIALIZER`
- It must call `[super initWith...]` (the superclass's designated initializer)
- All other init methods must call `[self initWith...]`
- `init` inherited from `NSObject` should be overridden to call the designated initializer with sensible defaults

### Subclassing

When subclassing:
- Override the superclass's designated initializer
- If you add new state, declare a new designated initializer
- Clearly document which init is designated
- Call the super's designated initializer from yours

### Categories for Functionality

Categories are the preferred mechanism for decomposing a class into logical sections, but:
- Do not override methods in a category (use subclassing instead)
- Prefix category methods with a lowercase prefix to avoid collisions
- Each category for a "real" class should be in its own file (e.g., `GTMBook+GTMBookStorage.h` / `.m`)

### `dealloc` Placement

Place `dealloc` immediately after the initializers, before any other methods. It should only release non-ObjC resources (CF types, `malloc`'d memory, observer deregistration) since ARC handles ObjC ivars.

```objectivec
- (void)dealloc {
  [[NSNotificationCenter defaultCenter] removeObserver:self];
  if (_cFileHandle) {
    fclose(_cFileHandle);
  }
}
```

### `@autoreleasepool`

Use `@autoreleasepool` blocks to manage memory in tight loops or when creating many temporary objects. Do not use the old `NSAutoreleasePool` API.

```objectivec
for (NSInteger i = 0; i < 10000; i++) {
  @autoreleasepool {
    NSString *temp = [self generateLargeString];
    // process temp
  }
}
```

---

## 8. Objective-C++

### C++ within `.mm` Files

When writing Objective-C++ (`.mm` files), C++ code follows the Google C++ Style Guide. Objective-C code within the same file follows the Google Objective-C Style Guide. This dual-context means:

- C++ classes: `CamelCase` or `snake_case` per C++ guide
- C++ functions: `PascalCase` for non-static, or per project C++ conventions
- C++ member variables: trailing underscore `member_` (C++ convention) vs leading underscore `_ivar` (ObjC convention) — keep them visually distinct
- Avoid storing C++ objects in Objective-C containers; use `std::vector` / `std::map` instead
- C++ exceptions and ObjC exceptions are distinct — do not mix `@try` / `@catch` with C++ `try` / `catch`
- Use `.mm` extension, never `.m`, for files containing any C++

### Objective-C within C++

When using Objective-C types in C++ code, follow ObjC naming and conventions for the ObjC portions:

```cpp
#include <string>
#import "GTMBook.h"

class BookWrapper {
 public:
  void Open() {
    [book_ open];
  }

 private:
  GTMBook *book_;
};
```

---

## 9. Review Checklist

Before approving Objective-C code, verify:

- [ ] Prefixes are 3+ characters
- [ ] Class, protocol, category names use UpperCamelCase with prefix
- [ ] Method names are lowerCamelCase and read like sentences
- [ ] Variables are lowerCamelCase; ivars have leading underscore; globals have `g` prefix
- [ ] File-static constants use `k` prefix; public constants use project prefix
- [ ] All caps acronyms: URL, ID, TIFF, EXIF
- [ ] Line length ≤ 100 characters
- [ ] 2-space indentation, no tabs
- [ ] K&R brace style
- [ ] `@property` attributes are explicit (nonatomic/atomic, readwrite/readonly, strong/weak/copy/assign)
- [ ] ARC is used (no retain/release)
- [ ] Literals used for NSString, NSArray, NSDictionary
- [ ] `NS_ENUM` / `NS_OPTIONS` for enums and bitfields
- [ ] Designated initializers marked with `NS_DESIGNATED_INITIALIZER`
- [ ] `instancetype` used for init and class factory return types
- [ ] `NS_ASSUME_NONNULL_BEGIN` / `END` wraps headers
- [ ] Nullability annotations on all pointer parameters and return types
- [ ] No `#define` for constants (use `const` / `enum`)
- [ ] Lightweight generics on collection properties
- [ ] File names match primary class name
- [ ] Category methods have descriptive lowercase prefix
- [ ] `dealloc` only handles non-ObjC resources
- [ ] `@autoreleasepool` used in tight loops
- [ ] `.mm` files follow both C++ and ObjC style guides as appropriate

---

## Source

Full guide: `G:\styleguide\objcguide.md`
