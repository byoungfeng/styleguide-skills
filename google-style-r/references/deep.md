# Google R Style Guide — Deep Reference

## 1. Introduction

The Google R Style Guide is a fork of the Tidyverse Style Guide with Google-specific modifications. It defines coding conventions for R code at Google, covering naming, syntax, pipe usage, documentation, and general best practices. This guide should be used alongside the Tidyverse Style Guide, which acts as the baseline for any rules not explicitly overridden here.

The key motivation behind Google's modifications is to improve code readability and maintainability at scale, particularly in large codebases with many contributors. The most notable divergence from Tidyverse is the use of `BigCamelCase` for function names (rather than snake_case) and the requirement for explicit `return()` statements.

Full source: `G:\styleguide\Rguide.md`

---

## 2. Syntax

### 2.1 Naming Conventions

| Category | Convention | Example | Rationale |
|---|---|---|---|
| Functions | `BigCamelCase` | `CalculateMean()`, `LoadData()` | Distinguishes functions from objects; avoids ambiguity in large codebases |
| Private functions | `.` prefix | `.ValidateInput()` | Signals internal use; not exported |
| Objects (variables, columns, parameters) | snake_case | `mean_value`, `data_set` | Follows Tidyverse conventions; readability |
| S3 methods | dot.case | `print.my_class` | Required by R's S3 dispatch; however, avoid dot.case for other purposes |

**BigCamelCase rationale:** In the Tidyverse Style Guide, functions use snake_case (e.g., `mean()`). Google chooses `BigCamelCase` to visually distinguish function calls from object references. This is especially important in code where objects and functions operate on similar concepts.

**Private function prefix:** A leading dot (`.`) indicates the function is internal/private. This is a long-standing R convention. Example:

```r
.ReadAndValidateInput <- function(file_path) {
  data <- read.csv(file_path)
  .ValidateColumns(data)
}
```

**Avoid `dot.case` for non-S3 purposes:** Using dots in names (e.g., `load.data`) can conflict with S3 method dispatch. R treats `load.data` as an S3 method for class `data` on generic `load`. To avoid ambiguity, use snake_case for objects and BigCamelCase for functions.

```r
# Good
load_data <- read.csv("data.csv")
ProcessData <- function(data) { ... }

# Avoid — potential S3 conflict
load.data <- read.csv("data.csv")
```

### 2.2 `attach()` Prohibition

Never use `attach()`. It pollutes the global environment, creates subtle scoping bugs, and makes code unpredictable. Alternatives:

```r
# Good — reference explicitly
mtcars$mpg

# Good — use with() for temporary scope
with(mtcars, mean(mpg))

# Good — assign to a variable
mpg <- mtcars$mpg
```

```r
# Bad
attach(mtcars)
mean(mpg)  # ambiguous, fragile
```

### 2.3 Assignment Operators

Use `<-` for assignment. Do not use `=` for assignment (except for function parameters) and never use `->` (right-hand assignment).

```r
# Good
x <- 5
y <- dplyr::filter(data, column == 1)

# Acceptable — function parameter default
function(x = 10) { ... }

# Bad
x = 5
data %>% filter(column == 1) -> result
```

Right-hand assignment (`->`) makes code read left-to-right but assign right-to-left, which harms readability. Always assign to the left.

```r
# Good
result <- data %>%
  dplyr::filter(column == 1) %>%
  dplyr::mutate(new_col = column * 2)

# Bad
data %>%
  dplyr::filter(column == 1) %>%
  dplyr::mutate(new_col = column * 2) -> result
```

### 2.4 Explicit `return()`

Always use explicit `return()` statements. Do not rely on implicit returns (the last evaluated expression).

```r
# Good
CalculateMean <- function(x) {
  return(mean(x, na.rm = TRUE))
}

# Bad — implicit return
CalculateMean <- function(x) {
  mean(x, na.rm = TRUE)
}
```

Implicit returns can lead to subtle bugs, especially when functions grow or when control flow changes. An explicit `return()` makes the programmer's intent clear. This applies to all functions, including anonymous functions and closures.

```r
# Good
TransformData <- function(data, scale = FALSE) {
  if (scale) {
    return(scale(data))
  }
  return(data)
}

# Bad — missing returns in branches
TransformData <- function(data, scale = FALSE) {
  if (scale) {
    scale(data)  # implicit, fragile
  }
  data  # only returned if scale is FALSE
}
```

### 2.5 Namespace Qualification

Qualify all external function calls with their namespace using `::`. This makes dependencies explicit and avoids name collisions.

```r
# Good
result <- purrr::map(data, ~ .x * 2)
plot <- ggplot2::ggplot(data, ggplot2::aes(x = var1, y = var2))

# Bad
result <- map(data, ~ .x * 2)
```

#### Exceptions to `::` Qualification

| Exception | Reason | How to Handle |
|---|---|---|
| Infix functions (e.g., `%>%`, `%+%`) | `::` does not work with infix operators | Use `@import` in NAMESPACE |
| rlang pronouns (`.data`, `.env`) | Must be imported for pronoun behavior | Use `@import rlang` or `@importFrom rlang .data` |
| Base R packages (datasets, utils, grDevices, graphics, stats, methods) | Always available; low collision risk | May `@import` fully |

```r
# Infix — must use @import
@importFrom magrittr %>%
result <- data %>% dplyr::filter(column > 0)

# rlang pronoun — must be imported
@import rlang
my_var <- "mpg"
dplyr::filter(mtcars, .data[[my_var]] > 20)

# Base R — may @import
@import stats
```

---

## 3. Pipes

### 3.1 Left-to-Right Only

Pipes must always flow left-to-right. Right-hand assignment with `->` is prohibited.

```r
# Good — left-to-right
result <- data %>%
  dplyr::filter(column > 0) %>%
  dplyr::mutate(new_col = column * 2)

# Good — base R pipe
result <- data |>
  subset(column > 0) |>
  transform(new_col = column * 2)
```

```r
# Bad — right-hand assignment with pipe
data %>%
  dplyr::filter(column > 0) %>%
  dplyr::mutate(new_col = column * 2) -> result
```

### 3.2 Pipe Choice

Both `%>%` (magrittr) and `|>` (base R 4.1+) are acceptable. Choose based on context:

- `|>`: No extra dependency; simpler placeholder behavior (uses `_` in R 4.2+)
- `%>%`: Richer placeholder (`.`), works with R < 4.1, wider ecosystem support

```r
# Base R pipe
result <- data |>
  subset(column > 0) |>
  transform(new_col = column * 2)

# magrittr pipe
result <- data %>%
  dplyr::filter(column > 0) %>%
  dplyr::mutate(new_col = column * 2)
```

When using `%>%`, qualify with `@importFrom magrittr %>%` in the package documentation or use `@import magrittr`.

---

## 4. Documentation

### 4.1 Package-Level Documentation

Place package-level documentation in a file named `packagename-package.R`. This file should contain the `_PACKAGE` roxygen2 tag and any global package documentation.

```r
# mypackage-package.R

@keywords internal
"_PACKAGE"

## usethis namespace: start
## usethis namespace: end
NULL
```

### 4.2 Roxygen Tags — `@importFrom`

Place `@importFrom` tags in the Roxygen header of the function that uses the dependency. This keeps imports close to their usage and makes dependencies easier to track.

```r
# Good
#' Calculate the mean of a column
#'
#' @param data A data frame
#' @param column A column name
#' @return The mean value
#' @importFrom dplyr pull
CalculateColumnMean <- function(data, column) {
  values <- dplyr::pull(data, {{ column }})
  return(mean(values, na.rm = TRUE))
}
```

### 4.3 Avoid `@import`

Do not use `@import` to bring in all exported functions from a package (except for allowed base packages). `@import` creates a high risk of name collisions and makes code harder to audit.

```r
# Bad — brings in all dplyr functions
@import dplyr

# Good — specific imports
@importFrom dplyr filter mutate select
```

Allowed base packages that may be `@import`-ed fully:

- `datasets`
- `utils`
- `grDevices`
- `graphics`
- `stats`
- `methods`

---

## 5. Tidyverse Baseline

The Google R Style Guide inherits all rules from the Tidyverse Style Guide that are not specifically overridden. The following Tidyverse conventions apply:

### 5.1 File Naming

- File names should end in `.R`
- File names should be meaningful (e.g., `load-data.R`, `plot-results.R`)
- Avoid special characters in file names (use letters, numbers, hyphens, underscores)

### 5.2 Indentation

- Use two spaces for indentation
- No tabs

```r
# Good
MyFunction <- function(x, y) {
  result <- x + y
  return(result)
}
```

### 5.3 Spacing

- Spaces around all operators (`+`, `-`, `*`, `/`, `<-`, `==`, `>`, etc.)
- Space after commas, not before
- No space before parentheses in function calls
- Space before `{` in function definitions

```r
# Good
mean(x, na.rm = TRUE)
x <- 1:10
if (x > 5) {
  y <- x * 2
}

# Bad
mean(x,na.rm=TRUE)
x<-1:10
if(x>5){
  y<-x*2
}
```

### 5.4 ggplot2

Follow Tidyverse conventions for ggplot2:
- Each layer on its own line
- `+` at the end of the line
- Consistent aesthetic naming

```r
# Good
ggplot2::ggplot(data, ggplot2::aes(x = var1, y = var2)) +
  ggplot2::geom_point() +
  ggplot2::geom_smooth(method = "lm") +
  ggplot2::labs(title = "My Plot")
```

### 5.5 dplyr

Follow Tidyverse conventions for dplyr:
- Pipe chains for sequential operations
- One verb per line
- Arguments on separate lines when long

```r
# Good
result <- data %>%
  dplyr::filter(column > 0) %>%
  dplyr::group_by(category) %>%
  dplyr::summarise(
    mean_value = mean(value, na.rm = TRUE),
    n = dplyr::n()
  ) %>%
  dplyr::ungroup()
```

### 5.6 Comments

- Use `#` for comments
- Add a space after `#`
- Use `#` followed by `-----` for section breaks

```r
# This is a comment

# Section header -------------------------------------------

# Sub-section header
```

---

## 6. Review Checklist

Use this checklist when reviewing R code for Google style compliance.

### Naming
- [ ] Functions use `BigCamelCase`
- [ ] Private/internal functions prefixed with `.`
- [ ] Objects use snake_case
- [ ] No `dot.case` outside S3 methods

### Assignment
- [ ] `<-` used for all assignments
- [ ] `=` only in function parameter defaults
- [ ] No `->` (right-hand assignment)
- [ ] No `attach()`

### Returns
- [ ] `return()` is always explicit
- [ ] No implicit returns

### Namespace
- [ ] `::` used for all external functions
- [ ] Infix operators have `@import`
- [ ] rlang pronouns (`.data`, `.env`) are imported
- [ ] No `@import` for non-base packages
- [ ] `@importFrom` placed in function headers

### Pipes
- [ ] Left-to-right only
- [ ] No `->` with pipes

### Documentation
- [ ] Package-level doc in `packagename-package.R`
- [ ] `@importFrom` tags above the using function
- [ ] Minimal `@import` usage

### Tidyverse Baseline
- [ ] 2-space indentation (no tabs)
- [ ] Spaces around operators and after commas
- [ ] File names end in `.R` and are meaningful
- [ ] Line length ≤ 80 characters
- [ ] `#` comments with a space after `#`

---

Full source: `G:\styleguide\Rguide.md`
