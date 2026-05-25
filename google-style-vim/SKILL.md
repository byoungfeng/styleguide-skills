---
name: google-style-vim
description: Google Vimscript Style Guide — portability, naming, formatting, plugin structure, and best practices for Vim plugin development. Use when writing, reviewing, or refactoring Vimscript code to apply Google's coding standards. Triggers on any Vimscript coding task.
---

# Google Vimscript Style Guide — Quick Reference

## Portability

- **Strings**: Prefer single quoted (`'...'`) to avoid escape processing. Use double quotes only when you need special characters like `\n`, `\t`, or `"`.
- **Matching operators**: Always use case-sensitive (`=~#`, `!~#`) or case-insensitive (`=~?`, `!~?`) variants. Never bare `=~` / `!~` — they depend on the user's `ignorecase` setting.
- **Regex prefix**: Start all regex patterns with `\m\C` to force magic mode and explicit case sensitivity.
- **Dangerous commands**: Avoid `:s[ubstitute]`, `:g[lobal]`, `:bufdo`, `:windo`, `:tabdo` — they operate on unexpected buffers and can cause side effects.
- **Fragile commands**: Avoid `:map`, `:nmap`, `:vmap`, `:imap` (use `*map()` functions or `nmap <buffer>`). Avoid `:echo` (use `:echom` or `:echomsg`). Avoid `:exe` with user-supplied strings.
- **Catching errors**: Catch by error code (e.g., `catch /E123/`), never by error message text.

## General Guidelines

- **Messaging**: Communicate with the user infrequently. Use `echomsg` (not `echo`) so messages appear in `:messages`. Use `confirm()` or `input()` sparingly.
- **Type checking**: Use `is#` for strict type/identity checks. Prefer `maktaba#value#IsEqual()` for value comparisons over `==`.
- **Language preference**: Write in Vimscript unless there is a clear, measured performance reason to use Python. Avoid Lua, Ruby, Tcl, etc.
- **Boilerplate**: Use [maktaba](https://github.com/google/maktaba) for plugin boilerplate (setting management, error handling, logging).
- **Plugin layout**: Organize as:
  ```
  plugin-name/
    plugin/
      commands.vim
      autocmds.vim
      mappings.vim
    autoload/
      pluginname.vim
  ```

## Style

- **Indentation**: 2 spaces, no tabs.
- **Line width**: 80 characters max.
- **Continuation lines**: Indent +4 spaces from the opening line.
- **Spacing**: Spaces around operators (`=`, `+`, `==`, `is#`, etc.), after commas, after `:call` and `:return`.

## Naming

| Entity | Convention | Example |
|--------|-----------|---------|
| Plugin name | `plugin-names-like-this` | `vim-colorschemes` |
| Function name | `FunctionNamesLikeThis` | `GetCurrentLine()` |
| Command name | `CommandNamesLikeThis` | `FormatFile` |
| Augroup name | `augroup_names_like_this` | `filetype_detect` |
| Variable name | `variable_names_like_this` | `g:buffer_list` |

Always prefix variables with their scope: `g:`, `s:`, `a:`, `l:`, `v:`, `b:`, `w:`, `t:`.

## Full Source

`G:\styleguide\vimscriptguide.xml`
