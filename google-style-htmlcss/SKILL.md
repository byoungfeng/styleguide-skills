---
name: google-style-htmlcss
description: Google HTML/CSS Style Guide — formatting, naming, semantics, and best practices for HTML and CSS. Use when writing, reviewing, or refactoring HTML/CSS/Sass code to apply Google's coding standards. Triggers on any HTML or CSS coding task.
---

# Google HTML/CSS Style Guide — Quick Reference

## General

- **Protocol**: Use HTTPS for embedded resources
- **Indentation**: 2 spaces, no tabs
- **Capitalization**: All lowercase — HTML elements, attributes, CSS selectors, properties
- **Encoding**: UTF-8 (no BOM), specify `<meta charset="utf-8">` in HTML
- **Comments**: Explain code as needed, use `TODO:` for action items

## HTML

- **Document type**: `<!doctype html>` (no-quirks mode)
- **Validity**: Use valid HTML where possible
- **Semantics**: Use elements for their purpose (`<a>` for links, `<h1>` for headings)
- **Multimedia fallback**: Provide `alt` text for images, captions/subtitles for video
- **Separation of concerns**: Structure (HTML) vs presentation (CSS) vs behavior (JS)
- **Entity references**: Don't use `&mdash;` etc. — use actual UTF-8 characters. Only escape `<` and `&`
- **Optional tags**: Omit optional tags for file size (`<html>`, `<head>`, `<body>`, `<p>` closing, etc.)
- **`type` attributes**: Omit for stylesheets (`rel="stylesheet"`) and scripts (`src=""`)
- **`id` attributes**: Avoid unnecessary ids; use classes for styling, data attributes for scripting. If id required, include hyphen for JS safety
- **Formatting**: New line per block/list/table element, indent children
- **Quoting**: Double quotes `""` for attribute values

## CSS

- **Validity**: Use valid CSS
- **Class naming**: Meaningful/generic, short but descriptive, hyphen-separated, prefix for large projects
- **Selectors**: Avoid type-qualified classes (`.error` not `div.error`), avoid ID selectors
- **Shorthand**: Use shorthand properties where possible
- **Values**: Omit units after `0` (unless required), include leading `0`, use 3-char hex where possible
- **`!important`**: Avoid
- **Hacks**: Avoid user-agent detection and CSS hacks
- **Formatting**: Alphabetize declarations (optional), indent block content, semicolon after every declaration, space after colon, space before `{`, one selector per line newline-separated, blank line between rules
- **Quoting**: Single `''` quotes for attribute selectors and property values; no quotes in `url()`
- **Section comments**: Optional grouping with comments

## Sass / GSS

Also applies to Sass and GSS files.

---

Full source: `G:\styleguide\htmlcssguide.html`
