# Google Vimscript Style Guide — Deep Reference

## 1. Background

This guide is a casual, practical set of conventions for writing Vimscript that feels consistent, readable, and portable. It follows the spirit of Google's Vimscript style guide (the full XML source lives at `G:\styleguide\vimscriptguide.xml`). It is not a formal spec — it is a shared vocabulary so that every plugin looks like it was written by the same person.

The rules here exist because Vimscript is a wild language with decades of backward compatibility. Many constructs that _work_ will break for users with different settings, locales, or Vim versions. The goal is to write plugins that behave identically everywhere.

---

## 2. Portability Deep Dive

Vimscript's biggest trap is that user settings (`'ignorecase'`, `'magic'`, `'compatible'`, etc.) change how basic operations work. Portability means writing code that ignores whatever the user has configured.

### 2.1 Strings — Single vs Double

Always prefer single quoted strings:

```vim
let s:path = 'C:\Users\example'
let s:pattern = '\m\C\<word\>'
```

Single quotes treat every character literally — no escape sequences except escaping a single quote (`''`).

Double quoted strings interpret `\n`, `\t`, `\`, `"` etc.:

```vim
let s:msg = "line one\nline two"
```

Use double quotes only when you need escape sequences. If you don't need them, use single quotes — they are safer and more predictable.

### 2.2 Matching Operators

Never use bare `=~` or `!~`. These respect the user's `'ignorecase'` setting, which means the same regex can match or fail depending on the user's configuration.

Always use the explicit suffix:

| Operator | Meaning |
|----------|---------|
| `=~#` | Match, case-sensitive |
| `!~#` | No match, case-sensitive |
| `=~?` | Match, case-insensitive |
| `!~?` | No match, case-insensitive |

```vim
" GOOD: explicit case sensitivity
if l:name =~# '\m\C^foo'

" BAD: depends on &ignorecase
if l:name =~ '^foo'
```

The same rule applies to `==` vs `==#` / `==?` for string comparison, though `is#` (see section 3.2) is preferred for type-safe identity checks.

### 2.3 Regex Prefixing

Always prefix regex patterns with `\m\C`:

```vim
" GOOD
if l:text =~# '\m\Cfoo.*bar'

" BAD — depends on 'magic' setting
if l:text =~# 'foo.*bar'
```

- `\m` forces magic mode (the default, but explicit is safe).
- `\C` forces case-sensitive matching (overrides `'ignorecase'`).

Without these, a user with `:set nomagic` or `:set ignorecase` will get different behavior.

### 2.4 Dangerous Commands

The following commands should be **avoided** because they are unpredictable in a plugin context:

| Command | Why it's dangerous |
|---------|-------------------|
| `:s[ubstitute]` | Operates on the current line of the current buffer — depends on cursor position and user intent. |
| `:g[lobal]` | Can match unintended lines if `'magic'` or `'ignorecase'` differ. Also modifies the buffer in ways the user didn't request. |
| `:bufdo`, `:windo`, `:tabdo` | Iterate over all buffers/windows/tabs. Can cause massive side effects, change the user's layout, or crash on hidden buffers. |
| `:normal` (without `!`) | Executes normal mode commands that respect mappings. The user may have remapped keys. |

Prefer `:normal!` (with the bang) to avoid mapped keys:

```vim
" GOOD
normal! dd

" BAD — may run user mappings
normal dd
```

### 2.5 Fragile Commands

Many commands change behavior based on the user's environment:

- **`:map`** without `<buffer>` or without the `!` suffix — use `nmap <buffer>`, `vnoremap`, etc. or the `maktaba#plugin#Map` functions.
- **`:echo`** — use `:echomsg` so output is captured in `:messages` and doesn't interfere with command-line completion.
- **`:exe`** with user-supplied strings — this is a script injection vector. If you must use `:execute`, validate input.
- **`:redir`** — fragile and hard to get right. Use `maktaba#function#Get()` or capture output via `system()` when possible.

### 2.6 Catching Errors by Code

Vim's `:try` / `:catch` can match error messages, but messages are localized. A German or Japanese Vim will throw different message text.

Always catch by error code:

```vim
" GOOD
try
  call some#function()
catch /E123/
  echomsg 'Caught expected error'
endtry

" BAD — depends on locale
try
  call some#function()
catch /Undefined variable/
  echomsg 'Caught'
endtry
```

Error codes are numeric and stable across all locales and versions. Find the relevant code in `:help eval-errors` or by running the command and observing `:echo v:errmsg`.

---

## 3. General Guidelines Deep Dive

### 3.1 Messaging

Communicate with the user sparingly. Every `:echo`, `:echomsg`, or `:confirm` is an interruption.

```vim
" GOOD — messages are logged and non-blocking
echomsg 'Formatted ' . l:count . ' lines'

" BAD — blocking prompt
call confirm('Format ' . l:count . ' lines?', "&Yes\n&No")
```

Use `echomsg` (or `maktaba#log#Info`) over `echo` so output survives in the message history. Prefer `maktaba#log#*` functions when using maktaba.

### 3.2 Type Checking

Vim's `==` does type coercion. Use `is#` for strict identity:

```vim
" GOOD
if l:val is# 0
if l:val is# v:true

" BAD — '0' == 'false' in Vim
if l:val == 0
```

For value comparison (not identity), prefer `maktaba#value#IsEqual()`:

```vim
if maktaba#value#IsEqual(l:a, l:b)
  echomsg 'equal'
endif
```

### 3.3 Python (and Other Languages)

Write plugins in Vimscript. Use Python only when:

1. You have measured a performance bottleneck.
2. The task is impossible or prohibitively awkward in Vimscript (e.g., complex JSON parsing, HTTPS requests).

Never use Lua, Ruby, or Tcl — they add build-time and runtime dependencies for negligible benefit.

### 3.4 Plugin Layout

A well-structured plugin directory looks like this:

```
plugin-name/
  plugin/
    commands.vim       # :command definitions
    autocmds.vim       # :augroup / :autocmd definitions
    mappings.vim       # :nmap / :vmap definitions
  autoload/
    pluginname.vim     # functions, called on demand
  doc/
    pluginname.txt     # help file (optional but encouraged)
```

#### Functions — `autoload/`

Functions go in `autoload/` files so Vim loads them lazily. Always use `function!` (bang) and `abort`:

```vim
" GOOD — in autoload/pluginname.vim
function! pluginname#DoSomething(...) abort
  return 0
endfunction
```

- `!` allows reloading (essential during development).
- `abort` stops execution on the first error (without it, errors in the function body are silently ignored).

#### Commands — `plugin/commands.vim`

Define user-facing commands here. Do **not** use `!` — a plugin should only define each command once. If a user reloads, the command definition errors, which is preferable to silently overwriting.

```vim
" In plugin/commands.vim
command FormatFile call pluginname#FormatFile()
```

#### Autocommands — `plugin/autocmds.vim`

Wrap autocommands in an `:augroup` and clear the group first to avoid duplicates on reload:

```vim
" In plugin/autocmds.vim
augroup pluginname_autocmds
  autocmd!
  autocmd BufWritePre *.py call pluginname#PreWrite()
augroup END
```

The `autocmd!` clears all previous autocommands in the group, so reloading the script doesn't stack duplicates.

#### Mappings — `plugin/mappings.vim`

Use maktaba's `MapPrefix` or the `<Plug>` (not `<Leader>`) mechanism. Prefer `nmap <buffer>` / `vmap <buffer>` to scope mappings:

```vim
" In plugin/mappings.vim — using maktaba
call maktaba#plugin#Map('n', 'ga', ':call pluginname#Action()<CR>')

" Without maktaba — use noremap and <buffer>
nnoremap <buffer> ga :call pluginname#Action()<CR>
```

#### Settings — `plugin/settings.vim`

Use `:setlocal` (not `:set`) to avoid changing global values:

```vim
" GOOD — only affects the current buffer/window
setlocal tabstop=2
setlocal shiftwidth=2

" BAD — changes global default, affects all buffers
set tabstop=2
```

---

## 4. Style Deep Dive

### 4.1 Whitespace

- **Indent**: 2 spaces. No tabs.
- **Line width**: 80 characters maximum.
- **Continuation lines**: Indent +4 spaces from the start of the continued expression:

```vim
" GOOD
let s:long_string = 'this is a very long string that needs to be '
                  \ . 'continued on the next line'

" BAD — continuation at arbitrary indent
let s:long_string = 'this is a very long string that needs to be '
  \ . 'continued on the next line'
```

- **Spacing**: Spaces around operators:

```vim
" GOOD
let s:result = l:a + l:b
if l:x is# l:y
call some#func(l:arg1, l:arg2)

" BAD — no spaces
let s:result=l:a+l:b
if l:x is#l:y
call some#func(l:arg1,l:arg2)
```

- **Trailing whitespace**: Remove trailing whitespace from all lines **except** mappings (where a trailing `<Space>` or `<CR>` is intentional):

```vim
" Trailing space is OK here — it is part of the mapping
nnoremap <buffer> <Leader>w :write<Space>
```

### 4.2 Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Plugin (directory / repo) | `lowercase-with-hyphens` | `vim-colorschemes` |
| Function | `LeadingCaps` | `GetCurrentLine()`, `FormatBuffer()` |
| Command | `LeadingCaps` | `FormatFile`, `BuildProject` |
| Augroup | `snake_case` | `pluginname_autocmds` |
| Variable (global) | `snake_case` | `g:pluginname_option` |
| Variable (script) | `snake_case` | `s:cache_dict` |
| Variable (function arg) | `snake_case` | `a:filename` |
| Variable (local) | `snake_case` | `l:counter` |
| Variable (v:) | `snake_case` | `v:count` |

**Always prefix variables with their scope.** This is non-negotiable:

```vim
" GOOD
let g:pluginname_enabled = 1
let s:counter = 0
function! pluginname#Foo(a_filename) abort
  let l:lines = readfile(a:filename)
endfunction

" BAD — no scope prefix
let enabled = 1
let counter = 0
```

Omitting the scope prefix creates implicit global variables, which pollute the namespace and cause hard-to-find conflicts.

---

## 5. Review Checklist

Use this checklist when reviewing Vimscript code:

- [ ] Strings use single quotes by default; double quotes only when escapes are needed.
- [ ] All `=~` / `!~` operators use explicit `#` or `?` suffix.
- [ ] All regex patterns start with `\m\C`.
- [ ] No `:s[ubstitute]`, `:g[lobal]`, `:bufdo`, `:windo`, `:tabdo` used.
- [ ] `normal!` used (not `normal`).
- [ ] `:echomsg` used instead of `:echo` for user-facing messages.
- [ ] `:try` / `:catch` uses error codes, not message text.
- [ ] Functions declared with `!` and `abort`.
- [ ] Commands in `plugin/commands.vim` without `!`.
- [ ] Autocommands wrapped in `augroup` with `autocmd!`.
- [ ] Mappings use `<buffer>` or maktaba scoping; prefer `<Plug>` over `<Leader>`.
- [ ] Settings use `:setlocal` not `:set`.
- [ ] Indent is 2 spaces, no tabs.
- [ ] Lines ≤ 80 characters.
- [ ] Continuation lines indented +4.
- [ ] Spaces around operators.
- [ ] No trailing whitespace (except intentional in mappings).
- [ ] Naming follows conventions (snake_case variables, LeadingCaps functions/commands).
- [ ] All variables prefixed with scope (`g:`, `s:`, `a:`, `l:`, `v:`, `b:`, `w:`, `t:`).

---

Full source: `G:\styleguide\vimscriptguide.xml`
