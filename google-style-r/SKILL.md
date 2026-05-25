---
name: google-style-r
description: Google R Style Guide — naming, syntax, pipes, documentation, and best practices for R programming. Use when writing, reviewing, or refactoring R code to apply Google's coding standards. Triggers on any R coding task.
---

# Google R Style Guide — Quick Reference

Key differences from Tidyverse: This guide is a fork of Tidyverse with Google-specific modifications.

## Naming

- **Functions:** `BigCamelCase` (not snake_case) to distinguish from other objects
- **Private functions:** prefix with `.` (dot), e.g. `.DoSomethingPrivately`
- Moving away from `dot.case` (S3 method conflicts)
- **Objects:** snake_case (follow Tidyverse conventions)

## Syntax

- No `attach()` (creates errors)
- Use `<-` for assignment, not `=` (except for function parameters) and not `->` (right-hand assignment)
- Explicit `return()` — do NOT rely on implicit returns
- Qualify namespaces with `::` for all external functions (e.g., `purrr::map()`)
- Exceptions: infix functions `%name%` need `@import`, rlang pronouns like `.data` need import, base R packages (datasets/utils/grDevices/graphics/stats/methods) may be `@import`-ed fully

## Pipes

- No right-hand assignment with `->`
- Use `%>%` or `|>` pipe as preferred, but always left-to-right

## Documentation

- Package-level documentation in `packagename-package.R` file
- `@importFrom` tags in Roxygen header above the function using the dependency
- Avoid `@import` to bring in all functions (name collision risk)

## General

- Based on Tidyverse Style Guide for items not specifically mentioned
- Line length: 80 characters recommended
- Spaces around operators, after commas
- Use `#` for comments

Full source: `G:\styleguide\Rguide.md`
