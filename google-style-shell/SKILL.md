---
name: google-style-shell
description: Google Shell Style Guide — shebang, formatting, naming, quoting, and best practices for Bash scripting. Use when writing, reviewing, or refactoring shell scripts to apply Google's coding standards. Triggers on any shell scripting task.
---

# Google Shell Style Guide — Quick Reference

## Which Shell

- **Bash only** — `#!/bin/bash` at the top of every script
- Use `set` for options: `set -o errexit` (or `set -e`), `set -o nounset` (or `set -u`), `set -o pipefail`
- Common shebang: `#!/bin/bash`

## When to Use Shell

- Small utilities and simple wrappers only
- **< 100 lines** — if longer, rewrite in a more structured language (Python, Go, etc.)

## File Extensions

- Executables: `.sh` extension or no extension
- Libraries: must use `.sh` extension and should not be executable

## Naming

| Category | Convention | Example |
|---|---|---|
| Function names | `lowerCamelCase` or `_`-separated | `myFunc` or `my_func` |
| Variable names | `lowerCamelCase` | `myVariable` |
| Constants / env vars | `UPPER_SNAKE_CASE` | `MY_CONSTANT` |
| Source filenames | `lowercase_with_underscores.sh` | `my_script.sh` |

## Formatting

- **2-space indent**, no tabs
- **80-character line limit**
- Pipelines: one per line if they don't fit on 80 chars
- `; then` and `; do` on same line as `if`/`for`/`while`
- `case` statement: 2-space indent for alternatives

```shell
if [[ -f "${file}" ]]; then
  process_file "${file}"
fi

for i in "${items[@]}"; do
  echo "${i}"
done

case "${fruit}" in
  apple)
    echo "pie"
    ;;
  banana)
    echo "bread"
    ;;
  *)
    echo "unknown"
    ;;
esac
```

## Quoting

- Always quote variables: `"${var}"`
- Use `$()` for command substitution (not backticks)
- Use `[[ ... ]]` for tests (not `[ ... ]`)
- Use `"$@"` not `$*`
- Quote strings that contain variables, spaces, or special characters

```shell
# Good
result="$(command)"
[[ -z "${name}" ]]
my_func "$@"

# Bad
result=`command`
[ -z $name ]
my_func $*
```

## Features and Best Practices

- **ShellCheck** — strongly recommended for all scripts
- **`local`** — always use `local` for function-scoped variables
- **`readonly`** — declare constants with `readonly`
- **Error messages** — send to STDERR; use an `err()` function
- **Avoid `eval`** — never use `eval`
- **Prefer builtins** — use Bash builtins over external commands where possible

```shell
err() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*" >&2
}

readonly MAX_RETRIES=5

process_file() {
  local file="$1"
  if [[ ! -f "${file}" ]]; then
    err "File not found: ${file}"
    return 1
  fi
}
```

Full source: `G:\styleguide\shellguide.md`
