---
name: google-style-go
description: Google Go Style Guide — naming, formatting, best practices, and style decisions. Use when writing, reviewing, or refactoring Go code to apply Google's coding standards. Triggers on any Go coding task.
---

# Google Go Style Guide — Quick Reference

## Style Principles (ordered by priority)

1. **Clarity** — code is read far more than written; optimize for the reader
2. **Simplicity** — prefer the least mechanism to accomplish a task
3. **Concision** — high signal-to-noise ratio; avoid unnecessary boilerplate
4. **Maintainability** — code should be easy to change; predictable names and structure
5. **Consistency** — when in doubt, follow the broader Go ecosystem; local consistency over global when the team agrees

## Naming

| Category | Convention | Good | Bad |
|---|---|---|---|
| Packages | Lowercase, single word, concise, no underscores or mixed case | `http`, `sql`, `big` | `HTTP`, `util`, `common`, `helper`, `string_utils` |
| Exported | MixedCaps | `MaxRetries`, `ParseRequest` | `max_retries`, `MAX_RETRIES`, `kMaxRetries` |
| Unexported | MixedCaps | `maxRetries`, `parseRequest` | `max_retries`, `mMaxRetries` |
| Receivers | Short 1-2 letter abbreviation of type | `r` for `Reader`, `tr` for `TokenReader` | `self`, `this`, `reader` |
| Constants | MixedCaps, no ALL_CAPS | `MaxPacketSize`, `StatusOK` | `MAX_PACKET_SIZE`, `kMaxPacketSize` |
| Getters | No `Get` prefix | `user.Name()` not `user.GetName()` | `GetName()` |
| Functions | Verb-like for actions, noun-like for returns | `Marshal`, `NewParser`, `Count` | — |

## Formatting

- `gofmt` enforces all formatting — no tab/space debate
- No fixed line length; prefer to not wrap unless it harms clarity
- `err != nil` check pattern (not `if err == nil { ... }`)

## Imports

Group order: stdlib → other (third-party) → local.

```go
import (
    "fmt"
    "io"
    "strings"

    "github.com/google/uuid"
    "golang.org/x/net/context"

    "yourmodule/internal/config"
    "yourmodule/internal/db"
)
```

Local import names use MixedCaps when the package path suffix doesn't match the intended name.

## Error Handling

- Always check `err != nil` explicitly
- Wrap errors with `fmt.Errorf("context: %w", err)`
- Sentinel errors use `var ErrX = errors.New("description")`
- Custom error types implement `Error() string` and optionally `Unwrap() error`
- Panics are reserved for truly unrecoverable situations (not normal error handling)

```go
var ErrNotFound = errors.New("record not found")

if err != nil {
    return fmt.Errorf("fetch user %d: %w", id, ErrNotFound)
}
```

## Testing

- Table-driven tests are preferred
- Use `t.Fatal`/`t.Fatalf` when the test cannot continue; `t.Error`/`t.Errorf` otherwise
- Test functions named `TestXxx` with signature `func TestXxx(t *testing.T)`
- Test helpers call `t.Helper()` as first line
- Use `t.Cleanup` for teardown instead of defer in helpers

```go
func TestParse(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int
        wantErr bool
    }{
        {"valid", "42", 42, false},
        {"empty", "", 0, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Parse(tt.input)
            if tt.wantErr {
                require.Error(t, err)
                return
            }
            require.NoError(t, err)
            require.Equal(t, tt.want, got)
        })
    }
}
```

## Concurrency

- Use `sync`/`atomic` for protecting state
- Use channels for communication (signaling, streaming, coordination)
- Prefer `sync.Mutex` over channel-based mutexes

## Source

- `G:\styleguide\go\guide.md`
- `G:\styleguide\go\decisions.md`
- `G:\styleguide\go\best-practices.md`
