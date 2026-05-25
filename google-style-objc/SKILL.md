---
name: google-style-objc
description: Google Objective-C Style Guide — naming, formatting, types, declarations, and Cocoa conventions. Use when writing, reviewing, or refactoring Objective-C/Objective-C++ code to apply Google's coding standards. Triggers on any Objective-C coding task.
---

# Google Objective-C Style Guide — Quick Reference

## Principles

- Optimize for the reader, not the writer
- Be consistent with existing code
- Be consistent with Apple SDK conventions
- Every style rule should pull its weight

## Naming

Follow Apple's Cocoa coding guidelines. All caps for acronyms: `URL`, `ID`, `TIFF`, `EXIF`.

| Construct | Convention | Example |
|---|---|---|
| File names | Reflect class name | `GTMBook.h`, `GTMBook.m`, `GTMBookAdapter.mm` |
| Prefixes | 3+ characters | `GTM`, `ABC`, `MYC` |
| Class names | UpperCamelCase + prefix | `GTMBook`, `GTMHttpRequest` |
| Category names | Prefix + method prefix (lowercase_) | `GTMBook (GTMBookStorage)` + `gtm_` methods |
| Protocol names | UpperCamelCase + prefix | `GTMBookDelegate` |
| Method names | lowerCamelCase, read like sentences | `- (void)bookDidOpen:(GTMBook *)book;` |
| Functions | PascalCase + prefix (non-static) | `GTMBookOpenFile()`, static: `BookOpenFile()` |
| Variables | lowerCamelCase | `currentItem`, `didOpen` |
| Instance vars | Leading underscore | `_ivarName` |
| Globals | `g` prefix | `gGlobalVar` |
| Constants | Project prefix (public) or `k` prefix (file-static) | `GTMBookErrorDomain`, `kMaxCount` (file-static) |
| Enums | `NS_ENUM`, values with class prefix | `GTMBookTypePDF` |

## Formatting

- **Indent**: 2 spaces, no tabs
- **Column limit**: 100 characters
- **Brace style**: K&R (opening brace on same line)
- `@interface` / `@implementation`: opening brace on same line
- `@public` / `@private`: indented 2 spaces, colon aligned

```objectivec
@interface GTMBook : NSObject {
 @private
  NSString *_title;
}
@property(nonatomic, copy) NSString *title;
- (instancetype)initWithTitle:(NSString *)title;
@end
```

## Types

- Use `NSInteger` / `NSUInteger` / `CGFloat` for system API interactions
- Use `int32_t` / `int64_t` for exact-size values
- Avoid `unsigned int` for math — prefer `NSInteger`
- Use `NS_ENUM` / `NS_OPTIONS` macros for enums and bitfields

## Language Features

- **ARC is required**
- Prefer `@property` with auto-synthesize
- Use `NSString` / `NSArray` / `NSDictionary` literals: `@"foo"`, `@[@1, @2]`, `@{@"key": val}`
- No `+initialize` (use `+load` carefully if needed)
- Avoid `#define` for constants — use `const` or `enum`

## Cocoa Patterns

- **Designated initializers**: one init calls super's designated, others call self's designated
- Annotate with `NS_DESIGNATED_INITIALIZER`
- Use `NS_UNAVAILABLE` for forbidden init methods

## Source

Full guide: `G:\styleguide\objcguide.md`
