# Google Shell Style Guide — Deep Reference

Full source: `G:\styleguide\shellguide.md`

---

## 1. Background

### Which Shell to Use

- **Bash is the only shell language permitted.**
- Shebang must be `#!/bin/bash` — do not use `/bin/sh` or other shells.
- Use `set` for shell options at the top of every script:
  - `set -o errexit` (or `set -e`) — exit on command failure
  - `set -o nounset` (or `set -u`) — error on undefined variables
  - `set -o pipefail` — propagate errors through pipelines

```shell
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail
```

Some projects combine all three into a single line for brevity but the expanded form is preferred for readability.

```shell
set -euo pipefail
```

### When to Use Shell

- Shell is appropriate for **small utilities and simple wrappers**.
- If a script exceeds **100 lines**, it should likely be rewritten in a more structured language such as Python or Go.
- Shell scripts are not suitable for:
  - Complex data manipulation
  - Performance-critical tasks
  - Programs requiring data structures beyond arrays/associative arrays
  - Large codebases that need testing and maintainability

### 100-Line Limit Recommendation

- The 100-line limit is a guideline, not a hard rule.
- If you find yourself approaching it, refactor early.
- Consider splitting into multiple scripts or migrating to a different language.

---

## 2. Shell Files

### File Extensions

- **Executables** — should have `.sh` extension, or **no extension** at all.
- **Libraries** — must have `.sh` extension and should **not** be executable.
- Libraries are meant to be sourced, not executed directly.

```shell
# Executable script
# my_script.sh or just my_script
#!/bin/bash

# Library script — must be .sh, not executable
# my_lib.sh
```

### SUID/SGID

- **SUID and SGID are forbidden in shell scripts.**
- The kernel ignores the SUID bit on interpreted scripts for security reasons.
- Even if it worked, the security implications are too dangerous.
- If you need elevated privileges, use `sudo` or a dedicated binary.

---

## 3. Environment

### STDOUT vs STDERR

- **Normal output** goes to STDOUT.
- **Error messages** must go to STDERR.
- This allows callers to separate actual output from diagnostic messages.

```shell
# Bad — error goes to STDOUT
echo "Error: file not found"

# Good — error goes to STDERR
echo "Error: file not found" >&2
```

### `err` Function Pattern

Define a reusable `err()` function for consistent error reporting:

```shell
err() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*" >&2
}

if [[ ! -f "${configFile}" ]]; then
  err "Configuration file missing: ${configFile}"
  exit 1
fi
```

- Always include a timestamp for logging.
- The function sends all arguments to STDERR.
- Use `return` for function errors, `exit` for fatal script errors.

---

## 4. Comments

### File Header

Every script should start with a header comment describing its purpose:

```shell
#!/bin/bash
#
# Package build script.
#
# Takes a version number and builds the distributable package.
# Usage: ./build_package.sh <version>
```

### Function Comments

Functions should be documented with sections for Globals, Arguments, Outputs, and Returns:

```shell
#######################################
# Cleanup temporary build artifacts.
#
# Globals:
#   BUILD_DIR
#   TEMP_DIR
# Arguments:
#   None
# Outputs:
#   Status messages to STDOUT
# Returns:
#   0 on success, 1 on failure
#######################################
cleanup() {
  local result=0
  if [[ -d "${BUILD_DIR}" ]]; then
    rm -rf "${BUILD_DIR}" || result=1
  fi
  if [[ -d "${TEMP_DIR}" ]]; then
    rm -rf "${TEMP_DIR}" || result=1
  fi
  return "${result}"
}
```

- Globals — lists global variables read or modified
- Arguments — describes each positional parameter
- Outputs — what is printed and where (STDOUT/STDERR)
- Returns — exit codes or return values

### Implementation Comments

- Comment tricky, non-obvious, or interesting parts of the code.
- Do not restate what the code clearly does.
- Use comments to explain **why**, not **what**.

```shell
# We need to sort here because find(1) does not guarantee order
files=()
while IFS= read -r -d '' f; do
  files+=("${f}")
done < <(find "${dir}" -type f -print0)
IFS=$'\n' files=($(sort <<<"${files[*]}"))
unset IFS
```

### TODO Comments

- Use `TODO` with the developer's name or username in parentheses:

```shell
# TODO(jsmith): Handle the edge case where the input file is empty.
```

- TODOs should be actionable and specific.
- Do not leave unresolved TODOs in committed code without tracking them.

---

## 5. Formatting Deep Dive

### Indentation

- **2 spaces** per indent level.
- **No tabs.** Configure your editor to convert tabs to spaces.
- Nested constructs increase the indent level.

```shell
if [[ -f "${config}" ]]; then
  if [[ -r "${config}" ]]; then
    source "${config}"
  else
    err "Config file not readable: ${config}"
  fi
fi
```

### Line Length

- Maximum **80 characters** per line.
- If a line exceeds 80 characters, break it.
- For long strings, use heredocs or embedded newlines.

```shell
# Long string — use heredoc
longMessage=$(cat <<EOF
This is a very long message that would exceed the 80 character limit
if we tried to keep it on a single line in the script.
EOF
)

# Command with many arguments — break with backslash
compile \
  --input "${sourceDir}/main.c" \
  --output "${buildDir}/main.o" \
  --optimize 2 \
  --verbose
```

### Pipelines

- Pipelines that exceed 80 characters should be split one per line.
- Place the `|` at the **end** of the previous line.

```shell
# Short pipeline — single line
command1 | command2

# Long pipeline — one per line
command1 \
  | command2 \
  | command3 \
  | command4

# Alternative — no backslash needed when pipe is at line end
command1 |
  command2 |
  command3
```

### Control Flow — `; then`, `; do`, `; else`

- Place `; then` on the same line as `if`.
- Place `; do` on the same line as `for`/`while`/`until`.
- `; else` goes on the same line as `}`.

```shell
# Good
if [[ -f "${file}" ]]; then
  process "${file}"
fi

if [[ -f "${file}" ]]; then
  process "${file}"
else
  err "Missing: ${file}"
  exit 1
fi

for i in "${items[@]}"; do
  echo "${i}"
done

while read -r line; do
  echo "${line}"
done < "${input}"

# Bad
if [[ -f "${file}" ]]
then
  process "${file}"
fi
```

### `case` Statement Formatting

- 2-space indent for the alternatives.
- The `;;` terminator is at the same indent as the pattern.
- Use `esac` at the same indent as `case`.

```shell
case "${word}" in
  help)
    show_help
    ;;
  version)
    show_version
    ;;
  *)
    err "Unknown command: ${word}"
    exit 1
    ;;
esac
```

- Simple one-liner patterns may share the line:

```shell
case "${quiet}" in
  true|1) exec >/dev/null 2>&1 ;;
esac
```

### Variable Expansion

- Always **brace-delimit** variable names: use `"${var}"` not `"$var"`.
- Braces are required when the variable is adjacent to other characters:

```shell
# Brace required — otherwise $app_version would look for variable app_version
version="${app}_version"

# Brace optional but preferred for consistency
name="${USER}"
```

- Use the full set of Bash parameter expansion features:

```shell
# Default values
"${var:-default}"
"${var:=default}"
"${var:+alternate}"

# String substitution
"${var#prefix}"
"${var%suffix}"
"${var/pattern/replacement}"

# Length
"${#var}"

# Indirect reference
"${!var}"
```

### Quoting

#### Always quote variables

```shell
# Good — handles spaces, empty strings, special chars
echo "${filename}"

# Bad — word splitting and glob expansion
echo $filename
```

#### Use `"$@"` not `$*`

```shell
# Good — preserves arguments with spaces
for arg in "$@"; do
  echo "${arg}"
done

# Bad — merges all arguments into one string
for arg in $*; do
  echo "${arg}"
done
```

#### Single quotes for literal strings

```shell
# No variable interpolation or escaping needed
echo 'This is a literal string with $dollar signs'

# Single quotes inside single quotes — impossible, use double quotes
# instead or escape with '\''
```

#### Double quotes for strings with variables

```shell
# Variables and special chars are expanded
greeting="Hello, ${name}!"
echo "${greeting}"
```

#### Quoting command substitution

```shell
# Good — quoting the outer assignment protects against weird output
output="$(command)"

# Quoting inside $( ) is also needed for inner variables
result="$(grep "${pattern}" "${file}")"
```

---

## 6. Features and Bugs Deep Dive

### ShellCheck

- **Use ShellCheck** for all scripts.
- ShellCheck catches:
  - Unquoted variables
  - Missing `#!/bin/bash`
  - `eval` usage
  - Common portability issues
  - Logic errors
- Run it as part of CI or as a pre-commit hook.
- Suppress false positives with directives:

```shell
# shellcheck disable=SC2086
some_command $unquoted_var
```

### Command Substitution — Use `$(command)`

- `$(command)` is preferred over backticks.
- Backticks are harder to read and nest poorly.

```shell
# Good
currentDate="$(date +%Y-%m-%d)"

# Bad
currentDate=`date +%Y-%m-%d`

# Nesting is clear with $()
outer="$(command1 "$(inner_command)")"

# Nesting with backticks is a nightmare
outer=`command1 \`inner_command\``
```

### `[[ ... ]]` vs `[ ... ]` vs `test`

- Use `[[ ... ]]` for all test conditions.
- `[[ ... ]]` is a Bash keyword (not an external command) and has advantages:
  - No word splitting or glob expansion on variables
  - Supports `&&` and `||` inside the brackets
  - Supports regex matching with `=~`
  - Supports pattern matching with `==`

```shell
# Good — Bash-native, safe
if [[ "${name}" == "root" ]]; then
  echo "Running as root"
fi

if [[ "${number}" =~ ^[0-9]+$ ]]; then
  echo "Numeric"
fi

# Bad — external command, subject to word splitting
if [ "${name}" = "root" ]; then
  echo "Running as root"
fi
```

### Testing Strings

- Use `-z` to test for empty string.
- Use `-n` to test for non-empty string (the default behavior of `[[ ]]`).

```shell
# String is empty
if [[ -z "${var}" ]]; then
  echo "Variable is empty or unset"
fi

# String is non-empty
if [[ -n "${var}" ]]; then
  echo "Variable has a value"
fi

# Equivalent (but less explicit) — just test the string
if [[ "${var}" ]]; then
  echo "Variable has a value"
fi
```

### Wildcard Expansion

- Wildcard expansion (`*`, `?`, `[...]`) should be intentional.
- Quote variables to prevent accidental globbing.
- Use `set -o noglob` (or `set -f`) to disable globbing temporarily.
- Pattern matching in `[[ ... ]]` does not expand globs on the right side of `==` when quoted.

```shell
# Without quotes — glob expansion occurs
for file in "${dir}"/*; do
  echo "${file}"
done

# With quotes — literal asterisk, no expansion
echo "${var}"

# Pattern matching in [[ ]] — unquoted pattern works
if [[ "${filename}" == *.txt ]]; then
  echo "Text file"
fi
```

### Eval Prohibition

- **`eval` is forbidden.**
- `eval` executes arbitrary strings as code, leading to:
  - Command injection vulnerabilities
  - Hard-to-debug quoting issues
  - Unpredictable behavior
- Use arrays, associative arrays, or indirect references instead.

```shell
# Bad — never do this
eval "command ${userInput}"

# Better — use an array
args=()
args+=("${userInput}")
command "${args[@]}"

# For variable indirection, use ${!var}
varName="myVar"
echo "${!varName}"  # prints value of $myVar
```

### Arrays

Bash supports indexed and associative arrays:

```shell
# Indexed array
files=()
files+=("file1.txt")
files+=("file2.txt")
echo "${files[0]}"
echo "${#files[@]}"

# Associative array (Bash 4+)
declare -A config
config["host"]="localhost"
config["port"]=8080
echo "${config["host"]}"

# Always use "${array[@]}" to expand all elements
for f in "${files[@]}"; do
  echo "${f}"
done
```

### Pipes to `while`

Using a pipe to feed `while read` creates a **subshell**. Variables set inside the loop are lost:

```shell
# Bad — count is lost after the loop
count=0
find . -type f | while IFS= read -r file; do
  count=$((count + 1))
done
echo "${count}"  # prints 0

# Good — use process substitution
count=0
while IFS= read -r file; do
  count=$((count + 1))
done < <(find . -type f)
echo "${count}"  # prints correct count

# Also good — use a here-string
while IFS= read -r line; do
  process "${line}"
done <<< "${multiLineVariable}"
```

### Arithmetic

- Use `(( ... ))` for integer arithmetic.
- Use `$(( ... ))` for arithmetic expansion.

```shell
# Arithmetic evaluation — no $ needed for variables
i=0
((i++))
((i += 5))

if (( i > 10 )); then
  echo "Greater than 10"
fi

# Arithmetic expansion — use $ to capture result
result=$(( i * 2 ))
echo "${result}"
```

- Avoid `let` and `expr` — they are obsolete.

### Aliases

- **Aliases are not recommended in scripts.**
- Aliases only expand in interactive shells unless `shopt -s expand_aliases` is set.
- Use functions instead of aliases for reusable behavior.

```shell
# Bad — alias (won't expand in non-interactive scripts)
alias ll='ls -la'
ll /tmp

# Good — function
ll() {
  ls -la "$@"
}
ll /tmp
```

---

## 7. Naming Conventions Deep Dive

### Function Names

- **LowerCamelCase** or **underscore-separated** (choose one style and stay consistent).
- Function names should be descriptive verbs or verb-noun pairs.
- Lowercase with underscores is the more common style in the Google guide.

```shell
# LowerCamelCase
myFunction() {
  local arg="$1"
}

# Lowercase with underscores
my_function() {
  local arg="$1"
}

# Descriptive verb-noun
get_config_value() {
  local key="$1"
}
```

- All functions should define `()` after the name — the `function` keyword is optional.
- The opening brace goes on the same line as the function name with a space.

```shell
# Good
my_func() {
  echo "hello"
}

# Acceptable but redundant
function my_func() {
  echo "hello"
}
```

### Variable Names

- **LowerCamelCase** for regular variables.

```shell
local fileName=""
local maxCount=10
local outputDir=""
```

- Loop variables should be simple and descriptive:

```shell
for file in "${files[@]}"; do
  process "${file}"
done

for (( i = 0; i < 10; i++ )); do
  echo "${i}"
done
```

### Constants and Environment Variable Names

- **UPPER_SNAKE_CASE** for constants and environment variables.
- Declare constants with `readonly` at the top of the script or function.

```shell
# Global constants
readonly VERSION="1.0.0"
readonly CONFIG_PATH="/etc/myapp/config"
readonly MAX_RETRIES=5

# Inside a function — should still be readable
my_func() {
  readonly LOCAL_CONSTANT=42
}
```

- Environment variables set by the system (e.g. `PATH`, `HOME`, `USER`) are already uppercase — do not reassign them unless absolutely necessary.

### Source Filenames

- **Lowercase with underscores** for script and library filenames.
- Library files must have `.sh` extension.
- Executable files may omit the extension.

```shell
# Good filenames
my_script.sh
build_package.sh
utils.sh
config.sh
deploy

# Bad filenames
MyScript.sh  # mixed case
my-script.sh  # hyphens (OK but underscores are convention)
movescript  # unclear
```

### Use `local` Variables

- Every variable inside a function should be declared `local`.
- This prevents accidental overwriting of global variables.
- `local` is not strictly a Bash builtin but is widely supported and safe.

```shell
# Good — all locals
process_file() {
  local file="$1"
  local tempFile
  tempFile="$(mktemp)"
  ...
}

# Bad — pollutes global scope
process_file() {
  file="$1"  # overwrites any global $file
  tempFile="$(mktemp)"
  ...
}
```

### Function Location

- Define all functions **at the top of the file**, before any executable code.
- This makes the script easier to read — the main logic is at the bottom.
- Functions are parsed before execution begins, so ordering within the function section does not matter.

```shell
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

# --- Functions ---

usage() {
  echo "Usage: $0 [options]"
}

process() {
  local arg="$1"
}

# --- Main ---

main() {
  process "$@"
}

main "$@"
```

### `main` Function Pattern

- Use a `main()` function for the script's entry point.
- This is not a requirement but is strongly recommended.
- A `main` function:
  - Limits the scope of temporary variables
  - Makes the script more testable
  - Clearly separates setup from execution
  - Allows calling `main` with `"$@"` to pass through script arguments

```shell
main() {
  local inputFile="$1"
  local outputDir="$2"
  local verbose=false

  if [[ "${inputFile}" == "" ]]; then
    err "Usage: $0 <input> [output]"
    exit 1
  fi

  process_file "${inputFile}" "${outputDir}"
}

main "$@"
```

---

## 8. Calling Commands

### Checking Return Values

- Always check return values of commands.
- Use `if` statements, `||`, or `set -e` (but `set -e` has edge cases).
- Never assume a command succeeded — check explicitly.

```shell
# Explicit check with if
if ! rm "${tempFile}"; then
  err "Failed to remove ${tempFile}"
  return 1
fi

# Using ||
rm "${tempFile}" || {
  err "Failed to remove ${tempFile}"
  return 1
}

# Using set -e (implicit — use with caution)
set -o errexit
rm "${tempFile}"  # script exits if this fails
```

- **Pitfall**: In a pipeline, only the last command's exit code is used unless `set -o pipefail` is set.

```shell
set -o pipefail
cmd1 | cmd2 | cmd3  # exits if any of cmd1, cmd2, or cmd3 fails
```

### Builtin vs External Commands

- Prefer Bash builtins over external commands when possible.
- Builtins are faster and avoid spawning a subprocess.

```shell
# Use builtins
echo "hello"           # builtin
printf "%s\n" "hello"  # builtin
[[ -f "${file}" ]]     # builtin (keyword)
source "${file}"       # builtin

# Avoid external equivalents
/bin/echo "hello"      # external — unnecessary
test -f "${file}"      # external — use [[ ]] instead
. "${file}"            # POSIX — 'source' is clearer
```

- But do not sacrifice readability for micro-optimizations.
- Use external tools when they are clearer or more robust.

```shell
# Good use of external tool
sort "${file}" | uniq

# Could be done in pure Bash with great effort — not worth it
```

---

## 9. Review Checklist

Use this checklist when reviewing shell scripts:

**Basics**
- [ ] Shebang is `#!/bin/bash`
- [ ] `set -o errexit`, `set -o nounset`, `set -o pipefail` are set
- [ ] Script is under 100 lines (or has a good reason not to be)

**Formatting**
- [ ] 2-space indentation, no tabs
- [ ] No lines exceed 80 characters
- [ ] `; then` / `; do` on same line as control keywords
- [ ] Pipelines broken one per line when long
- [ ] `case` statements properly indented

**Quoting**
- [ ] All variable expansions are quoted: `"${var}"`
- [ ] `"$@"` used instead of `$*`
- [ ] Command substitution uses `$(...)`, not backticks

**Tests**
- [ ] `[[ ... ]]` used instead of `[ ... ]` or `test`
- [ ] String tests use `-z` / `-n` explicitly

**Safety**
- [ ] No `eval` anywhere
- [ ] No SUID/SGID on scripts
- [ ] No aliases in script logic
- [ ] Arrays used instead of string concatenation for lists

**Functions**
- [ ] Function-scoped variables declared with `local`
- [ ] Constants declared with `readonly`
- [ ] Functions defined before `main`
- [ ] `main "$@"` used at end of script

**Error Handling**
- [ ] Error messages go to STDERR
- [ ] `err()` function or equivalent used consistently
- [ ] Return values of critical commands checked
- [ ] `set -o pipefail` enabled

**Style**
- [ ] Function names are `lowerCamelCase` or `_`-separated
- [ ] Variable names are `lowerCamelCase`
- [ ] Constants and env vars are `UPPER_SNAKE_CASE`
- [ ] Source filenames are `lowercase_with_underscores.sh`
- [ ] Libraries end in `.sh` and are not executable

**Tooling**
- [ ] ShellCheck run and all warnings addressed
- [ ] TODO comments are tracked and actionable

---

Full source: `G:\styleguide\shellguide.md`
