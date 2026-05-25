# Google Common Lisp Style Guide — Deep Reference

## 1. Background

### 1.1 Metaguide conventions

This style guide uses the RFC 2119 keywords:

- **MUST** / **MUST NOT**: absolute requirements.
- **SHOULD** / **SHOULD NOT**: strong recommendations; exceptions must have a good reason.
- **MAY** / **OPTIONAL**: truly optional.

Conventions apply to all new code written at Google unless an exemption is granted. Old code is not required to change, but reformatting is encouraged when substantial edits are made nearby.

### 1.2 Scope

Covers Common Lisp as defined by the ANSI standard and in common use at Google. Where implementations diverge, prefer behavior consistent with SBCL (the dominant implementation).

## 2. General Guidelines

### 2.1 Principles

- **Readability counts**: Code is read far more often than it is written.
- **Consistency**: Follow existing conventions within a file/project. When in doubt, match the surrounding code.
- **Clarity over cleverness**: Write explicit, straightforward code. Avoid macro magic unless it provides a clear readability win.
- **Minimize dependencies**: Think before adding a library dependency.

### 2.2 Priorities

1. Correctness
2. Clarity
3. Performance (only when measured and needed)
4. Brevity (last priority)

### 2.3 Architecture

- Decompose into small, composable functions.
- Separate data from operations on data.
- Use generic functions and CLOS for polymorphic behavior, not case statements on type.
- Avoid deep heirarchies of inheritance — prefer composition and mixins.

### 2.4 Libraries and external code

- Prefer widely-used, well-maintained libraries.
- Vendor third-party code only when necessary and clearly mark it.
- Wrap external libraries with thin abstraction layers to limit blast radius of API changes.

### 2.5 Development process

#### 2.5.1 Code review

All changes MUST be reviewed by at least one other person. Reviewers should verify:

- Style guide compliance
- Correctness and test coverage
- Appropriate abstractions
- No unnecessary complexity

#### 2.5.2 Tests

Code MUST have tests. Use `fiveam` or the project's established test framework.

- Unit tests for individual functions.
- Integration tests for subsystems.
- Regression tests for bug fixes.

#### 2.5.3 Warnings

Code MUST compile without warnings. Suppress unavoidable warnings with `#+sbcl sb-ext:ignore-...` declarations, not by disabling global warning settings.

## 3. Formatting

### 3.1 Spelling and abbreviations

- Use full English words — no cryptic abbreviations.
- Accepted exceptions: `db`, `url`, `html`, `xml`, `json`, `ui`, `id`, `io`, `key`, `val`, `tmp`, `ret`, `arg`.
- When in doubt, spell it out.

### 3.2 Line length

**MUST** limit lines to **100 characters**.

```lisp
;; Good
(defun compute-geodesic-distance (point-a point-b &key (epsilon 1e-7))
  "Return the geodesic distance between POINT-A and POINT-B."
  (let ((dx (- (x point-a) (x point-b)))
        (dy (- (y point-a) (y point-b))))
    (sqrt (+ (* dx dx) (* dy dy)))))
```

Break lines before operators, not after. Align continuation lines with the form's arguments.

### 3.3 Indentation

Indent like a properly configured GNU Emacs with `cl-indent`. No tabs — use spaces only.

```lisp
;; Correct — aligned with first argument
(defun compute-distance (point-a point-b
                         &key (epsilon 1e-7))
  ...)

;; Correct — loop body indented
(loop for x in list
      when (oddp x)
        collect x)
```

Key indentation rules:

- Body of `defun`, `defmacro`, `defmethod`, `let`, `labels`, `flet`, `if`, `when`, `unless`, `with-*` macros: **2 spaces**.
- Argument lists wrapping: align to the first argument, or 4 spaces if that would exceed line length.
- `loop` clauses: align under the first clause keyword.
- `cond` clauses: 2-space indent, body of each clause indented another 2.

### 3.4 File header and in-package

Every file starts with:

```lisp
;;;; package.lisp — Package definition for my-project core
;;;;
;;;; This file defines the public API surface.
;;;;
;;;; TODO(jsmith): Split into smaller packages as the project grows.

(in-package #:my-project.core)
```

The `in-package` form appears after the header, on its own line, with no blank line before it.

### 3.5 Vertical whitespace

**One blank line** between top-level forms.

```lisp
(defun foo ()
  ...)

(defun bar ()   ;; < exactly one blank line above
  ...)
```

Inside a `let` or `labels`, use blank lines sparingly — group related bindings.

### 3.6 Horizontal whitespace

- No spaces between the function name and opening paren in calls.
- No space between `(` and the first argument.
- One space between arguments.
- No space before `)`.
- No space around parentheses in read macros `#'`, `#(`, `#{`.

```lisp
;; Good
(defun foo (x y)
  (list x y))

;; Bad — extra spaces
(defun foo ( x y )
  ( list x y ))
```

### 3.7 No tabs

Tabs are forbidden. Configure your editor to insert spaces.

## 4. Documentation

### 4.1 Docstrings

Every function, macro, class, slot, variable, and constant MUST have a docstring.

```lisp
(defun find-user (user-id)
  "Return the USER object with the given USER-ID, or NIL if not found."
  ...)

(defclass user ()
  ((name
    :initarg :name
    :reader user-name
    :documentation "The user's display name.")
   (email
    :initarg :email
    :reader user-email
    :documentation "The user's email address."))
  (:documentation "A registered user of the system."))
```

Docstrings should:

- Be complete sentences starting with a capital letter and ending in a period.
- Describe "what" and "why", not "how".
- Use the third-person indicative ("Returns the user...", not "Return the user...").
- Mention the return value for functions.

### 4.2 Comment conventions (semicolons)

Use the four-level system:

| Level | Usage |
|-------|-------|
| `;;;;` | File headers, license blocks |
| `;;;` | Top-level section/group comments |
| `;;`  | Inside function/macro body comments |
| `;`   | End-of-line comments (rare) |

```lisp
;;;; -----------------------------------------------------------------
;;;; Data-access layer — all database interactions go through here.
;;;; -----------------------------------------------------------------

;;; User queries

(defun find-active-users ()
  (;; Build the query
   let ((query "SELECT * FROM users WHERE active = 1"))
    ...)                    ; <-- end-of-line comment, use sparingly
  ...)
```

### 4.3 Grammar and punctuation

- Docstrings and `;;;` comments are complete sentences with caps and periods.
- `;;` body comments can be sentence fragments (no period).
- `;` end-of-line comments are brief notes only.

### 4.4 TODO comments

Format: `TODO(name): <actionable description>`.

```lisp
;; TODO(jsmith): Handle the edge case where the user list is empty.
;; TODO(b/12345678): Optimize this query after the launch.
```

- The name can be a username, bug tracker ID, or team alias.
- The description MUST be actionable.
- Resolve TODOs before submitting unless they track known issues.

### 4.5 Domain-specific languages (DSLs) and regex

When embedding regex or other DSLs, comment the non-obvious patterns:

```lisp
(defun parse-email (string)
  ;; Matches: local-part@domain.tld
  ;; Local part allows alphanumerics, ., _, % +, and -
  ;; Domain must have at least one dot-separated component.
  (let ((regex "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"))
    (cl-ppcre:scan regex string)))
```

## 5. Naming

### 5.1 Case

All symbols are lowercase. Lisp is case-insensitive by default, but use lowercase for readability and tool compatibility.

### 5.2 Word separator

Use hyphens (`-`) to separate words. Never use slashes, underscores, or dots.

```lisp
;; Good
(defun get-user-name ...)
(defvar *max-retries* ...)

;; Bad
(defun get_user_name ...)
(defun get.user.name ...)
```

### 5.3 Denote intent, not content

Name variables by what they mean, not their type or structure.

```lisp
;; Good — describes meaning
(defparameter *user-cache* (make-hash-table :test #'equal))

;; Bad — describes implementation detail
(defparameter *hash-table-for-users* (make-hash-table :test #'equal))
```

### 5.4 Global earmuffs

| Pattern | Convention |
|---------|------------|
| `+name+` | Global constants (`defconstant`) |
| `*name*` | Global variables (`defvar`/`defparameter`) |
| `$name`  | Global special variables in external libraries (rare) |
| No earmuffs | Lexical variables |

```lisp
(defconstant +pi+ 3.141592653589793d0)
(defvar *debug-level* 2)
```

### 5.5 Predicates

Predicate functions end in `P` or `-P`.

```lisp
(defun numberp (x) ...)        ;; built-in style
(defun file-exists-p (path) ...)  ;; traditional style
```

Use `-P` for compound names (`file-exists-p`), bare `P` for single-word names (`numberp`). Be consistent within a project.

### 5.6 No library prefix in symbol names

Do not encode the library or package name in the symbol name.

```lisp
;; Bad — redundant prefix
(my-project.core:my-project.core-find-user ...)

;; Good
(my-project.core:find-user ...)
```

### 5.7 Packages — :use vs explicit prefix

- Prefer not to use `:use` for other packages. Access external symbols with the explicit package prefix.
- Exception: utility packages used pervasively (e.g., `alexandria`, `serapeum`) where `:use` is acceptable.
- Use `:import-from` for selective imports.

```lisp
;; Preferred
(defpackage #:my-project.core
  (:use #:cl)
  (:import-from #:alexandria #:when-let #:if-let))

;; Acceptable for pervasive utility libs
(defpackage #:my-project.util
  (:use #:cl #:alexandria))
```

### 5.8 Don't shadow CL symbols

NEVER shadow symbols from the `COMMON-LISP` package. If you must extend a CL function, use a different name or `:import-from` with rename.

```lisp
;; Forbidden — shadows cl:list
(defpackage #:my-project.core
  (:use #:cl)
  (:shadow #:list))

;; Correct — rename on import
(defpackage #:my-project.core
  (:use #:cl)
  (:import-from #:alexandria #:compose))
```

### 5.9 No double-colon in production

Do not use `::` to access private symbols in another package in production code. If you must, document why with a comment. Tests are an acceptable exception.

```lisp
;; Bad — bypasses package interface
(my-project.internal::secret-function ...)

;; Good — documented exception in tests
;; We reach into the internal package here because no public API exists.
(my-project.internal::setup-test-fixture ...)
```

## 6. Language Usage

### 6.1 Mostly functional style

Prefer pure functions (no side effects). Isolate side effects to well-defined boundaries.

```lisp
;; Good — pure function
(defun compute-total (prices)
  (reduce #'+ prices :initial-value 0))

;; Acceptable — side effect at system boundary
(defun print-report (report stream)
  (with-slots (title lines) report
    (format stream "~&~A~%~{~A~%~}" title lines)))
```

### 6.2 Recursion vs iteration

Prefer iteration over recursion unless the recursive form is clearly cleaner.

```lisp
;; Preferred — iterative
(defun count-matching (predicate list)
  (loop for element in list
        when (funcall predicate element)
          count it))

;; Acceptable — natural recursion (tree walk)
(defun tree-depth (tree)
  (if (atom tree)
      0
      (1+ (reduce #'max (mapcar #'tree-depth tree)))))
```

Recognize when the compiler will tail-call optimize; do NOT rely on it without verifying (SBCL does; other implementations may not).

### 6.3 Loop and Iter

Use `loop` (the ANSI-standard macro) or `iter` (the `iterate` library). Avoid writing manual `do`/`tagbody`/`go` constructs.

```lisp
;; loop — good
(loop for x in list
      when (evenp x)
        sum x)

;; iter — also good (with iterate library)
(iter (for x in list)
      (when (evenp x)
        (sum x)))
```

### 6.4 CLOS style

- Use `defclass` / `defmethod` for polymorphism, not `typecase` or `etypecase` on data.
- Prefer `:reader` and `:writer` over `:accessor` unless you need both.
- Use `with-slots` and `with-accessors` to reduce repetition.
- Avoid `:initform` with non-constant values — use `:initarg` and `initialize-instance :after` instead.

```lisp
(defclass document ()
  ((title
    :initarg :title
    :reader document-title
    :documentation "The document title.")
   (body
    :initarg :body
    :reader document-body
    :documentation "The document body."))
  (:documentation "A text document."))

(defmethod render ((doc document) (target (eql :html)))
  (format nil "<h1>~A</h1><p>~A</p>"
          (document-title doc)
          (document-body doc)))
```

## 7. Review Checklist

Before submitting, verify:

- [ ] All functions have docstrings.
- [ ] All top-level forms are separated by exactly one blank line.
- [ ] No line exceeds 100 characters.
- [ ] No tabs in the file.
- [ ] No lonely parentheses on their own line.
- [ ] No shadowing of CL symbols.
- [ ] No `::` usage in production code.
- [ ] Package declaration is clean (`:use` minimal).
- [ ] Global constants use `+...+`, globals use `*...*`.
- [ ] Predicates end in `P` or `-P`.
- [ ] All symbols are lowercase with hyphens.
- [ ] No library prefix in symbol names.
- [ ] Code compiles without warnings.
- [ ] Tests pass.
- [ ] No TODOs without tracked issues.

---

Full source: `G:\styleguide\lispguide.xml`
