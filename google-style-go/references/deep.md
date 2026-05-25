# Google Go Style Guide — Deep Reference

This document synthesizes the three Google Go style guide documents: guide.md (core style rules), decisions.md (specific style decisions with reasoning), and best-practices.md (guidance on common patterns). Use this reference when writing, reviewing, or refactoring Go code.

---

## 1. Style Principles

### Clarity
The single most important goal. Code is read far more often than it is written. Prioritize readability over cleverness.

- **What vs. Why**: Comments should explain *why* something is done, not *what* the code does. The code itself should make the *what* obvious.
- **Self-documenting code**: Choose descriptive names so the code reads clearly without comments.
- **Avoid over-commenting**: If the code is clear, don't add a redundant comment.

```go
// Good: explains why, not what
// Use a non-blocking send so the caller is not stalled.
select {
case ch <- msg:
default:
}

// Bad: explains what (already obvious from code)
// Send msg to ch, or drop if full.
select {
case ch <- msg:
default:
}
```

### Simplicity
Prefer the least mechanism needed to accomplish a task.

- Don't introduce abstractions (interfaces, types, layers) before they earn their weight.
- A simple `for` loop is better than a complex generic or channel pipeline when the loop suffices.
- Avoid premature abstraction; wait for three or more concrete uses before extracting an interface.

```go
// Good: simple loop
var sum int
for _, v := range values {
    sum += v
}

// Over-engineered: unnecessary abstraction for a simple sum
type Summable interface {
    Add(int)
    Value() int
}
```

### Concision
High signal-to-noise ratio. Every line should earn its place.

- Avoid boilerplate and repetition.
- Favor `switch` over long `if`-`else if` chains.
- Prefer `range` loops with `_` for unused values explicitly discarded.

```go
// Good: concise switch
switch t := v.(type) {
case *Foo:
    return t.Bar()
case *Baz:
    return t.Quux()
default:
    return nil, fmt.Errorf("unexpected type %T", v)
}
```

### Maintainability
Code should be easy to change without introducing bugs.

- Predictable naming means readers can find code by guessing names.
- Avoid exported functions/variables unless they are needed outside the package.
- Keep functions small and focused on a single responsibility.

### Consistency
When in doubt, follow the broader Go ecosystem. When local conventions differ, local consistency may override global consistency only with team agreement and clear documentation.

- If the standard library uses a pattern, follow it.
- If the surrounding package uses a naming convention, follow it.

---

## 2. Core Guidelines

### Formatting with gofmt

`gofmt` (or `go fmt`) enforces all formatting rules. There is no debate about tabs vs. spaces (tabs for indentation), alignment, or brace style. Always run `gofmt` before committing.

```bash
gofmt -w .
```

### MixedCaps

Identifiers use MixedCaps (also called CamelCase) rather than underscores. This applies to all exported and unexported names.

```go
// Good
var maxRetries int
const StatusOK = 200
func ParseRequest(ctx context.Context, r io.Reader) (*Request, error)

// Bad
var max_retries int
const STATUS_OK = 200
func parse_request(ctx context.Context, r io.Reader) (*Request, error)
```

### Line Length

Go has no fixed line length limit. If a line is long, consider whether it can be refactored into shorter, clearer pieces — but do not mechanically wrap lines at a column limit. Leave long lines if breaking them would harm clarity.

```go
// Acceptable: long but clear
if err := db.WithTransaction(ctx, func(tx *sql.Tx) error { return tx.ExecContext(ctx, query, args...) }); err != nil {
    return fmt.Errorf("transaction failed: %w", err)
}

// Worse: forced wrapping that obscures structure
if err := db.WithTransaction(
    ctx,
    func(tx *sql.Tx) error {
        return tx.ExecContext(ctx, query, args...)
    },
); err != nil {
    return fmt.Errorf("transaction failed: %w", err)
}
```

### Naming

- Package names are lowercase, single word, concise, and describe the package's purpose.
- Exported names use MixedCaps and are accessible outside the package.
- Unexported names use MixedCaps and are package-private.
- The greater the distance between a declaration and its use, the more descriptive the name should be.

### Local Consistency

When code within a package or project follows a convention that differs from this guide, prefer local consistency. Update the convention to match the guide when possible.

---

## 3. Naming Deep Dive

### Underscore Exceptions

Underscores are permitted only in specific contexts:

1. **Generated code** — protobuf, `stringer`, etc. Do not hand-write underscores.
2. **Test function names** — to embed spaces in `go test -run` patterns.

```go
func TestParse_invalidInput(t *testing.T)  // acceptable in tests
```

3. **Low-level cgo or OS integration** — when interfacing with C code that uses underscores.

Everywhere else, use MixedCaps.

### Package Names

Rules and recommendations:

- Lowercase only, no underscores, no MixedCaps.
- Single word is strongly preferred. If a compound name is unavoidable, do not separate the words.
- Short and concise: `strconv` not `string_conversion`.
- Describe the package's purpose, not its contents: `http` serves HTTP, `json` implements JSON encoding.
- Use the filename-style name (the last element of the import path) as the package name.
- **Do not use** generic package names: `util`, `common`, `helper`, `base`, `lib`, `utils`. These become dumping grounds for unrelated functionality.

```go
// Good package names
package strings
package http
package sql

// Bad package names
package string_utils
package common
package helper
```

### Import Renaming

When renaming an import, the local name must follow MixedCaps (first letter lowercase for unexported-like names, uppercase for exported-like — typically lowercase).

```go
import (
    fpb "google.golang.org/grpc/health/grpc_health_v1"
    // used as fpb.HealthCheckRequest
)
```

### Blank Imports

Blank imports (`import _ "pkg"`) are permitted only for:

- `init()` side effects (e.g., database drivers, image formats).
- `go:linkname` references.
- Use with restraint and document why the blank import is needed.

### Dot Imports

Dot imports (`import . "pkg"`) are **strongly discouraged** except in tests, and only for `iotest` or similar packages where it is conventional.

### Receiver Names

- Short: 1 or 2 letters.
- An abbreviation of the receiver type.
- Consistent across all methods on the same type.
- Do not use generic names like `self`, `this`, or `me`.

```go
// Good
func (r *Reader) Read(p []byte) (n int, err error)
func (r *Reader) Close() error

func (tr *TokenReader) Next() (Token, error)
func (tr *TokenReader) Close() error

// Bad
func (this *Reader) Read(p []byte) (n int, err error)
func (self *Reader) Close() error

// Inconsistent receiver names across methods
func (tr *TokenReader) Next() (Token, error)
func (r *TokenReader) Close() error
```

### Constant Names

- MixedCaps, not ALL_CAPS.
- No `k` prefix (common in other languages).
- Group related constants with `iota` when appropriate.

```go
// Good
const MaxPacketSize = 1500
const StatusOK = 200
const (
    StateIdle State = iota
    StateBusy
    StateDone
)

// Bad
const MAX_PACKET_SIZE = 1500
const STATUS_OK = 200
const kMaxPacketSize = 1500
const (
    STATE_IDLE State = iota
    STATE_BUSY
    STATE_DONE
)
```

### Function and Variable Naming

- **Noun-like for functions that return values**: `Count()`, `Parser()`, `NewRequest()`.
- **Verb-like for functions that perform actions**: `Marshal()`, `Parse()`, `Write()`.
- **Avoid repetition**: don't repeat the package name in the function name.
- **No `Get` prefix**: `user.Name()` not `user.GetName()`.
- **No `Set` prefix**: `user.SetName()` is fine since there is no alternative noun form.

```go
// Good
user.Name()
user.ID()
user.SetName("Alice")

// Bad
user.GetName()
user.GetID()
```

### Acronyms

Acronyms keep consistent case. All caps for all-uppercase, all lowercase for all-lowercase.

```go
var userID int
const URL = "https://example.com"
func ParseHTTPRequest(r *http.Request)
```

An acronym at the start of an unexported name is lowercase: `userID`, `urlString`.

### Shadowed Package Names

Avoid naming variables or parameters that shadow package names.

```go
// Bad: shadows the "path" package
func Parse(path string) {
    // "path" is now shadowed
}

// Good: use a different parameter name
func Parse(p string) {
}
```

---

## 4. Formatting

### gofmt

Run `gofmt` on all Go code. It handles:

- Indentation (tabs, no spaces)
- Brace placement (opening brace on same line)
- Blank lines
- Import block sorting (by path)
- Alignment of comments, struct fields, etc.

Do not manually fight `gofmt`. If `gofmt` produces poor output, restructure the code.

### Comment Formatting

- `//` line comments are preferred over `/* */` block comments.
- `/* */` is reserved for package-level doc comments and disable-code.
- Doc comments on exported declarations should be complete sentences.
- Use `//` even for multi-line comments.

```go
// Package http provides HTTP client and server implementations.
package http

// Parse parses a request from r and returns a Request value.
func Parse(r io.Reader) (*Request, error) {
    // ...
}
```

### Line Length

Do not impose a fixed line length limit. Break long lines only when it improves readability.

- If a line exceeds 80-100 columns, consider refactoring it into multiple statements.
- Function signatures can use multiple lines if the parameter list is long.
- Don't wrap lines at an arbitrary column just because they look long.

```go
// Long but clear — no wrapping
return fmt.Errorf("serialization failed for type %T with options %+v: %w", t, opts, err)

// Better to refactor than to wrap
resp, err := client.DoRequest(ctx, method, url, headers, body, timeout)
if err != nil {
    return nil, fmt.Errorf("request failed: %w", err)
}
```

### Indentation

Use tabs (one tab per level). `gofmt` enforces this. Do not use spaces for indentation.

---

## 5. Import Decisions

### Import Grouping

Imports are grouped into at least two, sometimes three groups, separated by blank lines:

1. **Standard library** packages (stdlib)
2. **Other** (third-party, golang.org/x, etc.)
3. **Local** (the module's own packages, including internal)

Within each group, imports are sorted by path. `gofmt` handles the sorting.

```go
import (
    "context"
    "fmt"
    "io"
    "net/http"
    "os"
    "strings"

    "cloud.google.com/go/storage"
    "github.com/google/uuid"
    "google.golang.org/api/option"
    "google.golang.org/grpc"

    "yourmodule/internal/auth"
    "yourmodule/internal/db"
    "yourmodule/internal/server"
)
```

### Renaming Imports

Rename imports when the package name conflicts with a local name or is ambiguous.

```go
import (
    "crypto/rand"
    "math/rand"
    "encoding/json"

    "yourmodule/internal/rand" // ambiguous — rename
)

import (
    "crypto/rand"
    "math/rand"

    jwriter "yourmodule/internal/jsonwriter" // rename to avoid confusion
)
```

### Blank Imports

Use blank imports sparingly:

```go
import _ "image/gif"  // register GIF decoder
import _ "github.com/lib/pq"  // register PostgreSQL driver
```

Document why a blank import is needed.

### Dot Imports

Only used in test files, and only for packages where it is conventional (e.g., `iotest`).

```go
import . "io/iotest"
```

Do not use dot imports in production code.

---

## 6. Declarations

### `var` vs. `:=`

- `:=` is used inside functions when the type is clear from the right-hand side.
- `var` is used when declaring a zero-value variable, a package-level variable, or when the type is not obvious from the RHS.

```go
// Good: type is obvious
s := "hello"
n := 42
m := make(map[string]int)

// Good: zero value, explicit type
var buf bytes.Buffer
var mu sync.Mutex

// Good: package-level
var Version = "1.0.0"

// Good: type not obvious from RHS
var r io.Reader = os.Stdin
```

### `new` vs. `&`

`new(T)` is equivalent to `&T{}` but `&T{}` is preferred because it makes the zero-value initialization explicit. Generally, prefer composite literals over `new`.

```go
p := &Point{}   // preferred
p := new(Point) // acceptable but less common
```

### Initializing Structs

- Use named fields when the struct has more than one field.
- Use unnamed fields only in simple, package-internal cases with very few fields.

```go
// Good: named fields
req := &Request{
    Method: "GET",
    URL:    url,
    Header: make(Header),
}

// Bad: unnamed — fragile and unclear
req := &Request{"GET", url, nil, make(Header)}
```

### Nil Slices

Nil and empty slices are treated the same in most contexts. Prefer `var` for a nil slice over `make` with zero length.

```go
var s []int           // nil, but len(s) == 0, safe to range over
s := make([]int, 0)   // non-nil, empty — only if nil is semantically different
```

---

## 7. Error Handling

### Returning Errors

Always check errors. Do not discard them unless explicitly intentional.

```go
if err != nil {
    return fmt.Errorf("open config: %w", err)
}
```

### Error Wrapping

Use `%w` in `fmt.Errorf` to wrap errors. This enables `errors.Is` and `errors.As` to work through the error chain.

```go
return fmt.Errorf("process request %d: %w", id, err)
```

Do not wrap errors with `%s` or `%v` when you want the caller to be able to unwrap them. Use `%s` for human-readable-only context:

```go
// Wrap (caller can unwrap)
return fmt.Errorf("read file: %w", err)

// Human-readable only (caller cannot unwrap)
return fmt.Errorf("read file: %v", err)
```

### Sentinel Errors

Define sentinel errors at package level using `var` and `errors.New`. Name them `ErrXxx` (exported) or `errXxx` (unexported).

```go
var ErrNotFound = errors.New("record not found")
var errInsufficientFunds = errors.New("insufficient funds")
```

Compare with `==` or use `errors.Is`:

```go
if errors.Is(err, ErrNotFound) {
    // handle
}
```

Do not use sentinel errors for dynamic conditions; use error types instead.

### Error Types

Custom error types implement `Error() string`. They may implement `Unwrap() error` or `Unwrap() []error` for unwrapping.

```go
type ValidationError struct {
    Field string
    Err   error
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on %s: %v", e.Field, e.Err)
}

func (e *ValidationError) Unwrap() error {
    return e.Err
}
```

Check with `errors.As`:

```go
var ve *ValidationError
if errors.As(err, &ve) {
    fmt.Println(ve.Field)
}
```

### Panics

- Use panics only for truly unrecoverable situations: programmer bugs, invariant violations, state corruption.
- Do not use panics for normal error handling (e.g., file not found, invalid input).
- Library code should generally not panic; return errors instead.
- `init()` functions may panic on unrecoverable initialization failure.

```go
// Acceptable panic: programmer bug
if ptr == nil {
    panic("process: ptr must not be nil")
}

// Bad: panic for normal error
if err != nil {
    panic(err) // don't — return the error
}
```

### Checking Errors

The standard pattern is `if err != nil { ... }`, not the positive check `if err == nil { ... return ... }`.

```go
// Good
if err != nil {
    return err
}
log.Println("success")

// Avoid
if err == nil {
    log.Println("success")
    return
}
// error handling here
```

### `errors.Is` and `errors.As`

- `errors.Is(err, target)` — checks if any error in the chain matches a sentinel.
- `errors.As(err, &target)` — finds the first error in the chain that can be assigned to target.

```go
if errors.Is(err, io.EOF) {
    // end of file
}

var netErr *net.OpError
if errors.As(err, &netErr) {
    // network operation error
}
```

---

## 8. Testing

### Table-Driven Tests

The preferred testing pattern. Define a slice of anonymous structs with input, expected output, and an optional name.

```go
func TestParseDuration(t *testing.T) {
    tests := []struct {
        name  string
        input string
        want  time.Duration
        err   bool
    }{
        {name: "seconds", input: "10s", want: 10 * time.Second},
        {name: "minutes", input: "5m", want: 5 * time.Minute},
        {name: "invalid", input: "xyz", err: true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseDuration(tt.input)
            if tt.err {
                require.Error(t, err)
                return
            }
            require.NoError(t, err)
            require.Equal(t, tt.want, got)
        })
    }
}
```

### Test Helpers

Test helpers are functions used within tests. They must call `t.Helper()` as the first line so that failure messages report the caller's line number.

```go
func mustParse(t *testing.T, input string) time.Duration {
    t.Helper()
    d, err := ParseDuration(input)
    if err != nil {
        t.Fatalf("ParseDuration(%q) = %v", input, err)
    }
    return d
}
```

### `t.Cleanup`

Use `t.Cleanup` for teardown instead of `defer` inside test helpers. This ensures cleanup runs even if the helper panics.

```go
func withTempDir(t *testing.T) string {
    t.Helper()
    dir := t.TempDir()
    t.Cleanup(func() {
        os.RemoveAll(dir)
    })
    return dir
}
```

### `t.Error` vs. `t.Fatal`

- `t.Error` / `t.Errorf` — report failure but continue the test. Use for non-fatal assertions.
- `t.Fatal` / `t.Fatalf` — report failure and stop the test immediately. Use when the test cannot proceed.

In table-driven tests, prefer `t.Fatal` inside the subtest (it only stops that subtest).

### Test Package

Test files belong in the same package (`package foo`) for white-box tests, or in `package foo_test` for black-box tests. Use `foo_test` for integration-style or external API tests.

```go
// foo_test.go — external test package
package foo_test

import (
    "testing"
    "yourmodule/foo"
)

func TestFoo(t *testing.T) {
    // tests using foo.SomeExportedName
}
```

### Examples

Example functions are both documentation and tests. They live in `example_test.go` files.

```go
func ExampleParse() {
    d, err := ParseDuration("10s")
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println(d)
    // Output: 10s
}
```

### Benchmarks

Benchmark functions are named `BenchmarkXxx(b *testing.B)`. Use `b.ResetTimer()`, `b.ReportAllocs()`, and `b.RunParallel` for advanced benchmarks.

```go
func BenchmarkParseDuration(b *testing.B) {
    for i := 0; i < b.N; i++ {
        ParseDuration("10s")
    }
}
```

---

## 9. Best Practices

### Function and Method Naming

- Do not repeat the package name in the function name.
- Do not repeat the receiver type in the method name.

```go
// Good — package "http", function "Get" → http.Get
resp, err := http.Get(url)

// Bad — repeats package name
resp, err := http.HTTPGet(url)

// Good — receiver is "r", method is "Read"
r.Read(p)

// Bad — repeats receiver type
r.ReaderRead(p)
```

### Test Doubles and Helpers

Name test doubles after the real type with a `Test` or `Fake` prefix when the type is exported. For unexported test doubles, a short descriptive name suffices.

```go
type TestStore struct{ /* ... */ }
type fakeClient struct{ /* ... */ }
```

Keep test double packages minimal. If a fake is shared across many packages, put it in a separate `testutil` package and note the dependency.

### Utility Packages

Do not create `util`, `common`, `helper`, `base`, or `utils` packages. These accumulate unrelated functionality and become unmaintainable. Each function should belong to a package named after its responsibility.

```go
// Bad: any function fits here
package util

func StringSliceContains(slice []string, s string) bool { ... }
func IntMin(a, b int) int { ... }

// Good: named by responsibility
package strutil

func Contains(slice []string, s string) bool { ... }

package mathutil

func Min(a, b int) int { ... }
```

### Context

- `context.Context` is the first parameter of functions that need it.
- Do not store contexts in structs; pass them explicitly.
- Name the parameter `ctx` by convention.

```go
func Fetch(ctx context.Context, url string) (*Response, error) {
    // ...
}
```

### Receiver Type

- If the method mutates the receiver, use a pointer receiver.
- If the receiver is a map, func, channel, or slice that is not re-sliced, a value receiver is fine.
- If the method is called on a value that must not be copied (e.g., `sync.Mutex`), use a pointer receiver.
- Be consistent: if any method on a type needs a pointer receiver, all methods on that type should use a pointer receiver.

```go
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}
```

### Concurrency

- Use `sync.Mutex` or `sync.RWMutex` to protect shared state.
- Use `sync/atomic` for simple counters and flags.
- Use channels for communication (signaling, streaming, coordination), not for mutual exclusion.

```go
// Good: mutex for state protection
var mu sync.Mutex
var cache = make(map[string]*Result)

func Get(key string) *Result {
    mu.Lock()
    defer mu.Unlock()
    return cache[key]
}

// Good: channel for communication
type Worker struct {
    jobs chan Job
    done chan struct{}
}

func (w *Worker) Run() {
    for job := range w.jobs {
        job.Process()
    }
    close(w.done)
}
```

Avoid channel-based mutexes (a channel of size 1 used as a lock). Use `sync.Mutex` instead.

### Performance

- Do not optimize prematurely. Write clear code first, then profile.
- Escape analysis and inlining are compiler concerns; write idiomatic Go.
- Avoid allocations in hot paths when profiling shows it matters.
- Use `*bytes.Buffer` over `string` concatenation in tight loops.
- Use `sync.Pool` for frequently allocated, short-lived objects (after profiling).

### Dependencies

- Minimize dependencies. Every import is a dependency to manage.
- Prefer the standard library over third-party packages.
- When using third-party packages, prefer stable, well-maintained ones.

---

## 10. Review Checklist

- [ ] `gofmt` has been run on all files
- [ ] Names follow MixedCaps convention (no underscores except tests/generated)
- [ ] Package names are lowercase, single word, and not `util`/`common`/`helper`
- [ ] Receiver names are short (1-2 letters) and consistent per type
- [ ] Constants use MixedCaps, not ALL_CAPS or `k` prefix
- [ ] Getters have no `Get` prefix
- [ ] Function names are verb-like (actions) or noun-like (returns) and avoid repetition with package/receiver
- [ ] Import groups follow: stdlib → other → local, with blank-line separators
- [ ] Errors are checked with `if err != nil`
- [ ] Errors are wrapped with `%w` when propagating
- [ ] Sentinel errors are named `ErrXxx` and defined with `errors.New`
- [ ] Panics are reserved for unrecoverable situations
- [ ] Tests use table-driven pattern with `t.Run`
- [ ] Test helpers call `t.Helper()` and use `t.Cleanup` for teardown
- [ ] Context is the first parameter, named `ctx`, not stored in structs
- [ ] Pointer receivers are used consistently for mutating or lock-containing types
- [ ] No `util`/`common`/`helper` packages exist
- [ ] Dependencies are minimized; standard library preferred

---

## Full Source

- `G:\styleguide\go\guide.md`
- `G:\styleguide\go\decisions.md`
- `G:\styleguide\go\best-practices.md`
